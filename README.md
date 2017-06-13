# kato: Keep a Tim Organised -- GTD in a filesystem

Life in a Filesystem

## Purpose

The purpose of this project is to define a filesystem under which GTD style
tasks can be filed; and queried using traditional UNIX tools. Maybe later
adding a web service or dedicated scripts (or FUSE?)

The whole idea is to __reduce the friction__ to _capturing_, _processing_ (and
_actioning_), _tracking_ and _reviewing_ tasks and projects.

## Initial FS layout

`/` is the base directory of the kato.

* `/capture/`
 * `/capture/new` endpoint for creating new files
* `/action/`
 * `/action/tags/` *computed-with-symlinks*
 * `/action/tags/at-{computer,mobile,home}/` *computed-with-links*
 * `/action/tags/agenda/[email]/` *computed-with-links*
 * `/action/tags/anywhere/` *computed-with-links*
* `/waiting-on/`
* `/waiting-for/`
 * `/waiting-for/[email]/` *maybe*
* `/read/`
* `/tickle` *computed*
* `/reference/`
* `/someday-maybe/`
* `/review` *computed* 
* `/projects`
 * `/projects/GANTT` *computed*
 * `/projects/projname_1`
 * `/projects/projname_2`
 * `/projects/projname_2.d/projname_2.1`
 
## File Format

Files are to headed (optionally?) with meta-data. RFC2822 style -- NB 2822
section 3.4 for email address specification

Metadata fields:

| Field              | Format                                 | Description   |
|--------------------|----------------------------------------|---------------|
| `Title`            | string                                 | should be derived from filename |
| `Id`               | [[:alnum:]/\_-?.]+ (no space)          | hopefully unique (and short) identifier | 
| `Captured`         | 8601 Date                              | Date captured |
| `Last-Action`      | {NEXTACTION,REVIEWED,SOMEDAY,DONE,...} |               |
| `Last-Action-Date` | 8601 Date                              |               |
| `Schedule`         | 8601 Date range                        | scheduled/deadline or scheduled/ or /deadline |
| `Remind`           | 8601 Duration                          | Period between reminders on repeating commitments |
| `Project`          | string                                 | if not derived from containing folder |
| `Resources`        | list of _something(?)_                 | what a project might want -- no idea what format |
| `Actor`            | email                                  | for waiting-on |
| `Depends`          | id/title                               | for waiting-for |
| `Tags`             | list of tag                            | |
| `Location`         | Lat/long or description                | where the capture took place, probably |
| `Purpose`          | string                                 | strongly recommended or compulsory for Project files |
| `Values`           | string                                 | strongly recommended or compulsory for Project files |
| `Action-Energy`    | integer (%)                            | psychic energy required for the task |
| `Action-Time`      | ISO_8601-Duration (keep it to HH:MM)    | estimated time required for task |

If a metadata field is found, then a new line (blank line) separates the body
text from the header. Otherwise the whole file is body text.

Body text should be readable, but I wonder whether a markdown (GFM?) or SGML
would be less frictitious.

## Dates and Times

Dates and times will follow ISO_8601 as closely as possible. It means that
things like durations are a bit lumpy; but at least it's a common, unambiguous
standard.

* https://en.wikipedia.org/wiki/ISO_8601


### Dates: yyyy-mm-dd, yyyy-mm, yyyy 

__Abbreviations:__ yyyymmdd (no abbreviation for yyyy-mm)

A date specifies the field will be "active" (whatever that means for a field)
for the duration of that day, month or year. In calendars and tickles, these
will be "whole day" events (or reminders for all days in the period).

### Time: hh:mm:ss(zone)? hh:mm(zone)? hh(zone?)

__Abbreviations:__ hhmmss(zone?), hhmm(zone?)

Times (unless supported with another context e.g. "`today 1200`"), mean daily at
this time. They will be active every day at this instant.

### Date and Time: yyyy-mm-ddThh:nn:ss(zone)?

Hyphens or colons may be omitted:
* yyyy-mm-ddThh:nn:ss(zone)?
* yyyymmddThh:nn:ss(zone)?
* yyyy-mm-ddThhnnss(zone)?
* yyyymmddThhnnss(zone)?

__Abbreviations:__ yyyymmddThhmm(zone)?, yyyymmddThh(zone)?

Date and time together specify a point in time. There is no scope for truncating
the day or month from a date. 

### Durations: PnYnMnDTnHnMnS

__Abbreviations:__ the `P` must br provided, and then at least one further part
must be provided. If the duration is minutes only, it must be of the form `PTnM`
to avoid ambiguity with _n_ months.

A duration on its own used in the `Remind:` will become active after the
specified duration after `Last-Action-Date:`. That means if, for example:

> I have an action to put in my receipts with a `Remind: P7D`. I `kato-done` the
> item on a Tuesday. The next time it will be active is the following Tuesday.
> But if I `kato-done` it again on Thursday, the reminder will be moved to next
> Thursday.
>
> This would be different to `Remind: R/20170613/P7D` which would become active
> on the 20th June, then on the 27th June -- no matter when it is reset.

### Intervals: scheduled/deadline, scheduled/, /deadline

Intervals can specify _scheduled_ and a _deadline_ points in time.

___scheduled:___ is when the item becomes active. If it is omitted, then the
item is active from now until the deadline (and becomes overdue after that)

___deadline:___ is when the item becomes OVERDUE (marked with `†`) in calendar
reports.

ISO_8601 also specifies: _duration_/_datetime_ and _datetime_/_duration_; using
this will allow scheduled times to be calculated back and deadlines calculated
forward from the absolute _datetime_ provided.

### Recurrence: Rn/_interval_ R/_interval_

Recurs _n_ times or infinitely if n is not supplied.

Mostly, a count of the number of recurrences you missed will not be available;
just when the next (or previous) _n_ or so will be.

### Timezones

The timezone will be determined from the first of:

* timezone in _zone_
* `--timezone=[zone]` CLI switch
* `TZ` environment variable
* GMT

### :unicorn: Special Time Strings

Eventually we will handle special day/time strings accepted into the `kato`
command. But these will have to be normalised to their ISO_8601 equivalents (I
mean, what good is `today` as a `Captured` header?)

| Name | Meaning |
|------|---------|
| today | |
| now | |
| soon | now + 1H |
| tomorrow | today + 1D |
| teatime | next 1600 |


## `kato` Command Line Interface

__unstable__

```
kato cmd (notename) standard-options cmd-options
```

### Standard options:

| Short | Long | Default | Description |
|-------|------|---------|-------------|
|-d|--dir=|$KATOHOME| |
|-t'msg'|--text='msg'| | text that would otherwise come from -f file |
|-f _name_|--file=_name_ | - (stdin) | file for input |
|-e |--edit-after| false | run `kato-edit` afterwards |
|-v | --verbose | false | more output |
|-V | --version | | print version info and exit |
|   | --http | | print simple http headers |
|   | --format={json,text,html,ical} | text | change printer |

### Commands:

#### test
runs some tests

#### cards
produces cards pdf

#### init
#### sanity
* --schema
* --project-dependencies
* ...
#### tickle
* --today
#### tags
* --rebuild
* --list
* --search=
#### capture
#### delegate
#### defer
#### drop
* --no-archive
#### complete
* --no-archive
#### action / kato-do
`do` is not a good name for a shell command!
#### ical
* --push-url=
#### headers
##### add
##### remove
##### change
##### list
##### find
### agenda
* --for=_email-address_
### suggest
suggest a task given the curren context
* --max-energy=
* --max-time=

<!-- vim: cc=81 tw=80 et ts=2 sw=2
-->
