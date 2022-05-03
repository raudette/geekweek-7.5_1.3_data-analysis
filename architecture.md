# Application architecture for the pocket tag detector

We have, at hand, a proof of concept for a pocket detector of mobile location tracking devices one may have on themselves, as opposed to those carried by others. This detector consists of a Raspberry Pi computer hooked to a USB power pack, and running software that captures low-level Bluetooth beacons. It's a great basis to build a fuller-featured product on.

From that product, we require two main superfeatures:

1. A systematic data capture that enables analytics;
1. A data analysis system that would report occurrences of carrying a foreign tracker, either pulled on demand or pushed out as an *alert* of sorts.

This short document presents a software structure that supports the development of this product. Remark that this is just one instance of a suitable architecture, many others are possible, providing different trade-offs. The following describes first the data store that lives at the heart of the product. It then covers the data capture process. It closes on a pull-type monitoring interface. Along the way, we enumerate the technological pieces and the key aspects of code that glue the feature set together.

## The data store

As a rule of thumb, one should typically consider using a relational database as the data repository for a proof of concept. Unless one has expertise in alternative data stores, the relational database is a mature technology, with plenty of advice and tooling available on the Internet for building, using, profiling, debugging and maintaining. Pay no heed to the hype and use SQL. One can always migrate later to a database that suits the app's performance envelope better; smart people don't optimize before having proven a concept.

In addition, one can go a long way using the simplest relational database out there, and that is [SQLite](https://sqlite.org/index.html). The database will hold in a single file, making its migration to some other machine trivial. SQLite software builds on all major computing OS', it will even run out of tiny hardware, it's a no-brainer for a Raspberry Pi. From a performance perspective, it's also a strong baseline: it will fare well enough over many types of load and use cases.

The data schema for our product would be very simple. The only necessary element is a table to store all data one would retain about every Bluetooth beacon captured using the current software. Every beacon is associated to a device's identifier key, which in our software is named `manufacturerKeyHex`; the signal strength our model banks on for assessing nearness of the device is column `rssi`. One should also store the timestamp at which a beacon is received. The beacon is also associated with a list of identifiers under the name `deviceUUID`. If one wants to store these, they can be inserted in an another table, with a third table used to bind the beacon primary key to the device UUID primary key.

In Python, one uses the `sqlite3` module to interface with SQLite databases. Given that the database file lives in path `path`, one can use the context manager coding pattern to append new records to the database.

<a name="append-record"></a>
```python
import datetime as dt
with sqlite3.connect(path) as db:
    db.execute(
        "INSERT INTO beacons(timestamp, key, rssi) VALUES (?, ?)",
        (dt.datetime.now().isoformat(), record["manufacturerHexKey"], record["rssi"])
    )
```

Similarly, to get all beacons for key `key` between moments `tic` and `toc` (instances of class `datetime` from module `datetime`), one runs:

```python
with sqlite3.connect(path) as db:
    cursor = db.cursor()
    cursor.execute(
        """\
        SELECT * FROM beacons
        WHERE timestamp >= ?
        AND   timestap < ?
        AND   key = ?
        """,
        (tic, toc, key)
    )
    for timestamp, key, rssi in cursor:
        # Process each record as you please
```


## Data capture

At this point, one understands that the database is really the heart of this product. As such, data capture is simply a process that inserts records into the database as it senses new beacons. This process could be made to restart automatically in case of a crash.

A parallel process could be monitoring the database to analyze the beacons at regular intervals, or on cue from a table change. This process would load a short recent history of beacons and run a function of the form:

```python
def have_tracker_on_myself(beacons):
    return [key for key in beacons_inferred_to_come_from_keys_on_myself]
```

The process would call this function and whenever it returns a non-empty list, it would insert these keys and the corresponding timestamps of beacon occurrence and tracking detection into a new database table: the *alert table*. The whole thing involves just some Python scripting modifications around the existing sensor software.


## Monitoring interface

Once we have an alert table, building a HTML interface that queries the alert table is straightforward. One could serve it using the [Flask](https://flask.palletsprojects.com/en/2.1.x/) Python library, using a single RESTful route off of `localhost`. The interface could simply query the alert table and articulate it in an HTML report. Once this initial feature skeleton is done, further interesting features would be a way to dismiss known alerts through this interface, causing a full reload (it's fine not to have a Reactive Responsive Two-Way Server-Side Rendered Minified Javascript ball of CSS chicken wire and Javascript duct tape; that's a problem for quiche eaters), as well as populating and editing an exclusion list of, say, beacon keys one owns.

The interface can be composed out of run-of-the-mill HTML forms and controls: no need for a special GUI library unless one has expertise in such things. Here are the routes one could put together:

| Path | What it does |
|-----:|:-------------|
| `/` | Main interface. |
| `/dismiss?id=ID` | Clear a selected alert and return the main interface. |
| `/accept?key=KEY` | Add the key to the list we don't raise alerts when they beacon; return main interface. |


## Conclusion: **KEEP THINGS SIMPLE**

A proof-of-concept product is meant to prove a precise point. The point in question here is:

> I can walk around with a thing that will detect location trackers an adversary would hide on my person.

The point is proven already! We know we can capture this data, and we have an algorithm to process this data at such a small computation cost that it can be done off of the Raspberry Pi. Anything more is *communication*: you convince your audience of your ability to detect the trackers, and how that detection occurs in "real life." This communication is not helped by putting in fluff: fluff gets in the way of working software, and gets in the way of demonstrating your point.

Good communication is supported by software so simple that it cannot break, and will not obfuscate your point.
