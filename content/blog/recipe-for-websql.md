---
title: "A recipe for recreating the functionality of Web SQL"
type: post
date: 2021-02-09T12:00:00+08:00
---

*Firstly, let's step back briefly in history.*

[Web SQL](http://www.w3.org/TR/webdatabase/) was a way for javascript (and web apps) to use a relational database (specifically, [sqlite](http://sqlite.org)) that was stored in the web browser (ie. client side). It was popular amongst front-end developers for a while as it was a convenient and powerful method for storing and manipulating complex data in the browser.

Unfortunately, for various reasons, it didn't become a web standard and was discontinued in 2010.

Despite it being a long time ago now, every once in a while, someone will still lament that Web SQL had to die.

*Back to the present*

I write [Gig'o'Books](http://gigobooks.com) and the initial version of it stored user data in an sqlite file on the local computer. Since it was desktop software, this is nothing unusual.

Later, however, I wanted to port Gig'o'Books to the web, and so, I needed a way to run sqlite on the client-side web, in the browser. I found [sql.js](http://github.com/sql-js/sql.js) which, thanks to the magic of WebAssembly, works very well. Extremely well.

The web edition of Gig'o'Books managed user data in an sqlite database in the browser's javascript runtime. However, it wasn't persisted. Every time the user refreshed the browser, they needed to load their data anew by 'opening' a file on their local computer (assuming that they had saved the data previously). This was by design.

But it wasn't good enough. Subsequent user feedback indicated that the database did need to be persisted across browser sessions. Otherwise, it was too easy to accidentally lose data.

I looked around for an easy and simple way to persist the sqlite database and I found IDB-Keyval. [IDB-Keyval](http://github.com/jakearchibald/idb-keyval) allows simple key-value pairs to be stored in browser storage.

I experimented with storing the entire database (ie. as a serialised string/binary) as a single key-value. After playing around with it, performance seemed fine. It didn't feel noticeably slow or anything.

So I went with it.

<p style="text-align: center;">* * *</p>

After a while, I thought back about what had been done ... Wait, did I just recreate Web SQL? 🤔

Well, sort of. It's not exactly the same but it's good enough ... Actually. It's. Quite. Good.

So I guess ... Yes I did, I did just recreate Web SQL. Web SQL is alive!

<p style="text-align: center;">* * *</p>

So, what if you want to do this too?

Unfortunately, I don't have generic full working code to give out. The web edition of Gig'o'Books waits for the user to click 'recover' before loading the sqlite database from indexedDB and that's probably not what you want in your app.

But I can give you a recipe that I'm certain will work, and I can reveal some snippets from the source code of the web edition of Gig'o'Books. Implementing the rest is an exercise for the reader. 🙂

Tinkering with it is also encouraged. 🙂

## Recipe

1. Use sql.js to create an sqlite database and do stuff with it.
2. Whenever there is a change to the database, export it from sql.js and save a snapshot to indexedDB.
3. When the page or webapp is loaded by the browser, inspect indexedDB to see if there is a saved snapshot. If there is, then load the snapshot into sql.js.

Note that we are not writing and reading indexedDB for every change to the database. It's writing only when there is a change, yes, but reading only when the page or webapp is loaded. Furthermore, the writes can be throttled for better performance.

Effectively, indexedDB is an infrequently used mostly-write cache.

Further details are below.

Note that code snippets are from (the web edition of) Gig'o'Books verbatim and contain Gig'o'Books-specific references. They won't work out-of-the-box with other code. You'll have to extract out the important parts and integrate into your own code yourself.

Also, some code snippets are typescript instead of vanilla javascript.

### Detecting a change and calling a handler

Any query which starts with one of `['insert', 'del', 'delete', 'update']` and results in [.getRowsModified()](https://github.com/sql-js/sql.js/blob/36859494d53f1962f2059d9597782f5f4ab49172/src/api.js#L1092) returning positive constitutes a change.

```js
            // This code is from the knex dialect for sql.js
            // getRowsModified() is only applicable for insert, update and delete
            let rowsAffected = 0
            const lowerSql = obj.sql.trim().toLowerCase()
            ;['insert', 'del', 'delete', 'update'].forEach(op => {
                if (lowerSql.startsWith(`${op} `)) {
                    rowsAffected = obj.connection.getRowsModified()
                }
            })

            // Notify upstream if any rows were changed
            if (rowsAffected && !obj.options.skipOnChange && obj.sql.indexOf('knex_migrations_lock') == -1 && this.config.onChange) {
                this.config.onChange(obj)
            }
```


If there is a change, then trigger a call to a change handler:

```ts
    onChange() {
        ...

        // Trigger out of band: Since snapshot() interacts with the DB, it's not
        // good practice to have it occur within a DB-triggered handler
        // Throttle: If multiple changes happen, only call snapshot() once at the end
        if (this.timeoutId) {
            clearTimeout(this.timeoutId)
        }
        this.timeoutId = setTimeout(() => {
            snapshot()
            this.timeoutId = 0
        }, 1000)
    }
```

The `snapshot()` calls are throttled (`snapshot()` is defined below).

Since a single user-initiated action can result in multiple changes to the database, `snapshot()` is only called (ie. database is persisted to indexedDB) after all the changes are done. A debouncing period of 1000 milliseconds works well.

### Creating and loading snapshots

Here is the code from Gig'o'Books that interacts with indexedDB via IDB-Keyval.

```ts
// snapshot.ts
import { SqlJs } from 'sql.js/module'
import * as Idb from 'idb-keyval'
import { Project } from '../core'

const SNAPSHOT = 'snapshot'

export type Snapshot = {
    title: string
    filename: string
    timestamp: Date
    mru: string
    data: Uint8Array
}

export async function snapshot() {
    const database: SqlJs.Database = Project.database as any        // typecast

    const item: Snapshot = {
        title: Project.variables.get('title'),
        filename: Project.project!.filename,
        timestamp: new Date(),
        mru: window.location.hash.substring(1),
        data: database.export()
    }
    await Idb.set(SNAPSHOT, item)
}

export function getSnapshot(): Promise<Snapshot> {
    return Idb.get(SNAPSHOT)
}

export function deleteSnapshot(): Promise<any> {
    return Idb.del(SNAPSHOT)
}
```

Again, this is code directly from Gig'o'Books and won't work out of the box for your app. Don't worry about `Project` nor `.title`, `.filename`, `.timestamp` nor `.mru`. These are all Gig'o'Books-specific things.

The important field is `.data` which holds the database in serialised form.

You can see this source file is very very simple. It's basically just a thin wrapper around `Idb.set(), Idb.get(), Idb.del()`.

During page load or webapp initialisation, create a database from the snapshot (if exists) by doing something like:

```js
    const snapshot = getSnapshot()
    if (snapshot) {
        const database = SQL.Database(snapshot.data)
        ...
    }
```

### Knex

Some of the code snippets reference [knex](http://knexjs.org/) (which is a popular and useful database abstraction/ORM-like layer) because Gig'o'Books uses knex over sql.js.

If you use knex, please note that knex's built-in sqlite driver (which knex calls a 'dialect') is meant to be used against [node-sqlite3](http://github.com/mapbox/node-sqlite3)'s API and doesn't work with sql.js.

(I wrote a customised sql.js dialect for knex so Gig'o'Books could use it.)

## Demo and outcome

If you want to see (something very similar to) this recipe working, head to the [Gig'o'Books webapp](http://gigobooks.com/webapp) and play around with it.

Open up the Storage tab in your browser's developer tools and look at the indexedDB items.

Everytime you make a change, notice that:

* An asterisk appears next to the browser window/tab title (to indicate an unsaved change), and
* The snapshot that is cached in indexedDB changes.

How well does this work?

For Gig'o'Books, it works quite well (Feel free to try to break Gig'o'Books 🙂). Exporting into indexedDB is quite quick relative to humans, and also relatively infrequent. Performance feels good.

(I admit I didn't actually measure the overhead.)

If your webapp has a very large sqlite data file, maybe the overhead of export-and-persist might be too much. YMMV.

## Discussion and future directions

When I created this solution for myself, I didn't know about some existing alternatives:

* http://github.com/Philzen/WebSQL-Polyfill
* http://github.com/yathit/ydn-db

but I'm happy with this solution anyway. It's very simple to understand, it's transparent, and it's easy to both build and customise.

Unfortunately, I didn't have generic full working code to publish. It was either post about this without full code, or not post about it at all. I chose the former.

Possible future work:

* Package up the recipe into a generic module,
* Publish the knex dialect for sql.js that Gig'o'Books uses.

We'll see. Thanks for reading!

## One more thing

If you work with sqlite, remember to [VACUUM](https://sqlite.org/lang_vacuum.html) once in a while.
