# kato: Keep a Tim Organised -- GTD in a filesystem

Life in a Filesystem

## Purpose

The purpose of this project is to define a filesystem under which GTD style tasks can be filed; and queried using traditional UNIX tools. Maybe later adding a web service or dedicated scripts (or FUSE?)

The whole idea is to __reduce the friction__ to _capturing_, _processing_ (and _actioning_), _tracking_ and _reviewing_ tasks and projects.

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

Files are to headed (optionally?) with meta-data. RFC2822 style -- NB 2822 section 3.4 for email address specification

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
| `Action-Energy`    | %                                      | psychic energy required for the task |
| `Action-Time`      | ISO8601-Duration (keep it to HH:MM)    | estimated time required for task |

If a metadata field is found, then a new line (blank line) separates the body text from the header. Otherwise the whole file is body text.

Body text should be readable, but I wonder whether a markdown (GFM?) or SGML would be less frictitious.
