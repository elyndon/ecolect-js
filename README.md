# Ecolect

[![npm version](https://badge.fury.io/js/ecolect.svg)](https://badge.fury.io/js/ecolect)
[![Build Status](https://travis-ci.org/aholstenson/ecolect-js.svg?branch=master)](https://travis-ci.org/aholstenson/ecolect-js)
[![Coverage Status](https://coveralls.io/repos/aholstenson/ecolect-js/badge.svg)](https://coveralls.io/github/aholstenson/ecolect-js)
[![Dependencies](https://david-dm.org/aholstenson/ecolect-js.svg)](https://david-dm.org/aholstenson/ecolect-js)

Ecolect helps with parsing natural language to intents and values. This can be
used as a part in building a natural language interface for things such as bots,
voice or search interfaces.

## Installation

```
$ npm install --save ecolect
```

## Matching intents

The main function of Ecolect is to match natural language expressions to
intents. Every expression is parsed into tokens that are matched and scored
using a language specific comparison function. This allows the library to
match for example `cookies` even if the user skipped the last `s` and entered
`cookie` instead.

Matching can also be run in two modes:

* Normal mode - Match a full expression to an intent, which is best used when building a bot or a voice interface.
* Partial mode - Match most of the expression, for example when building an action launcher or a auto-complete for a search engine

### Example

```javascript
const ecolect = require('ecolect');
const en = require('ecolect/language/en');
const { any } = require('ecolect/values');

const intents = ecolect.intentsBuilder(en)
  .intent('lights:on')
    .value('room', any())
    .add('turn lights on')
    .add('turn lights in {room} on')
    .done()
  .intent('lights:off')
    .value('room', any())
    .add('turn lights off')
    .add('turn lights in {room} off')
    .done()
  .build();

// Normal mode - match the best
intents.match('turn lights off')
  .then(results => {
    if(results.best) {
      // One of the expressions matched
      console.log('Intent:', results.best.intent);
      console.log('Values:', results.best.values)

      // results.matches will contain the top matches if anything else matched as well
    }
  });

intents.match('turn lights', { partial: true })
  .then(results => {
    results.matches.forEach(match => console.log(match));
  });
```

## Options

Option                  | Default      | Description
------------------------|--------------|-------------
`partial`               | `false`      | If partial matching should be performed
`now`                   | `new Date()` | Date to use as a base for times and dates parsed
`weekStartsOn`          | `0` (Sunday) | The day the week starts on
`firstWeekContainsDate` | `1`          | The day of January which is always in the first week of the year.

### A note about weeks

It's important to set `weekStartsOn` and `firstWeekContainsDate` to something
expected by the user. The default value for `weekStartsOn` is `0` which
indicates that weeks start on Sunday.

`firstWeekContainsDate` defaults to `1` which is commonly used in North America
and Islamic date systems. Countries that use this week numbering include
Canada, United States, India, Japan, Taiwan, Hong Kong, Macau, Israel,
Egypt, South Africa, the Phillippines and most of Latin America.

For EU countries most of them use Mondays as the start of the week and the ISO
week system. Settings `weekStartsOn` to `1` and `firstWeekContainsDate` to `4`
will set weeks to a style used in EU and most other European countries, most
of Acia and Oceania.

Middle Eastern countries commonly use Saturday as their first day of week and
a week numbering system where the first week of the year contains January 1st.
Set `weekStartsOn` to `6` and `firstWeekContainsDate` to `1` to use this
style of week.

For more information about week numbering see the [Week article on Wikipedia](https://en.wikipedia.org/wiki/Week#Week_numbering).

## Values

Intents in Ecolect can also contain values, there are several built in types and
it's easy to provide custom value validation. Values are used to capture
information, such as dates, numbers, names and freeform text.

Values can either be used within intents or standalone as matchers:

```javascript
const en = require('ecolect/language/en');
const { date } = require('ecolect/values');

// Create a matcher for the date value
const dateMatcher = date().matcher(en);

// Call the matcher
dateMatcher('2018')
  .then(value => /* do something with the value */)
  .catch(err => /* handle errors */);

// Optionally specify options for parsing, such as what day the week starts on
dateMatcher('start of week 12', { weekStartsOn: 1 /* Monday*/ })
  .then(value => /* do something with the value */)
  .catch(err => /* handle errors */);
```

## Value types

### Integer

Capture any positive integer number.

Language         | Examples
-----------------|-------------
English          | `20`, `zero`, `one million`, `4 000`, `1 dozen`, `100k`

#### Returned value

The returned value is a simple object with one key named `value`.

```javascript
{ value: 2 }
```

#### Example

```javascript
const { integer } = require('ecolect/values');

builder.intent('list')
  .value('count', integer())
  .add('Show top {count} items')
  .done();
```


### Number

Capture any number, including numbers with a fractional element.

Language         | Examples
-----------------|-------------
English          | `20`, `2.4 million`, `8.0`, `-12`

#### Returned value

The returned value is a simple object with one key named `value`.

```javascript
{ value: 2.4 }
```

#### Example

```javascript
const { number } = require('ecolect/values');

builder.intent('add')
  .value('amount', number())
  .add('Add {amount} to result')
  .done();
```

### Ordinal

Capture an ordinal, such as `1st`, indicating a position.

Language         | Examples
-----------------|-------------
English          | `1st`, `third`, `3`, `the fifth`

#### Returned value

The returned value is a simple object with one key named `value`.

```javascript
{ value: 5 }
```

#### Example

```javascript
const { ordinal } = require('ecolect/values');

builder.intent('pick')
  .value('position', ordinal())
  .add('Show {position} in the list')
  .done();
```

### Date

Capture a date representing a single day.

Language         | Examples
-----------------|-------------
English          | `today`, `in 2 days`, `january 12th`, `2010-02-22`, `02/22/2010`, `first friday in 2020`

#### Returned value

The returned value is an object with the keys `year`, `month`, `day` and can
be turned into a `Date` via the function `toDate`.

```javascript
const date = value.toDate();
```

#### Example

```javascript
const { date } = require('ecolect/values');

builder.intent('deadline')
  .value('date', date())
  .add('Set deadline to {date}')
  .done();
```

### Time

Capture a time of day.

Language         | Examples
-----------------|-------------
English          | `09:00`, `3 pm`, `at 3:30 am`, `noon`, `quarter to twelve`, `in 2 hours`, `in 45 minutes`

#### Returned value

The returned value is an object with the keys `hour`, `minute`, `second` and can
be turned into a `Date` via the function `toDate`.

```javascript
const date = value.toDate();
```

#### Example

```javascript
const { time } = require('ecolect/values');

builder.intent('alarm')
  .value('time', time())
  .add('Wake me {time}')
  .done();
```

### Date & Time

Capture both a date and a time.

Language         | Examples
-----------------|-------------
English          | `3pm on Jan 12th`, `in 2 days and 2 hours`, `14:00`

```javascript
const { dateTime } = require('ecolect/values');

builder.intent('schedule')
  .value('when', dateTime())
  .add('Schedule a call {when}')
  .done();
```

### Date Interval

Capture an interval between two dates.

Language         | Examples
-----------------|-------------
English          | `today`, `this month`, `February to March`, `2018-01-01 to 2018-04-05`, `January 15th - 18th`

#### Returned value

The returned value is an object with two dates in the `start` and `end` keys.
Objects can be turned into dates with `toStartDate()` and `toEndDate()`:

```javascript
const start = value.toStartDate();
const end = value.toEndDate();
```

#### Example

```javascript
const { dateInterval } = require('ecolect/values');

builder.intent('add')
  .value('interval', dateInterval())
  .add('Todos with deadline within {dateInterval}')
  .done();
```

### Date Duration

Capture a duration.

Language         | Examples
-----------------|-------------
English          | `2 days`, `2m, 1d`, `1 year and 2 days`, `4y 2m`, `1 week`

#### Returned value

The returned value is an object containg fields with the change, such as
`years`, `weeks`, `months` and `days`. The function `toDate(currentTime)` can
be used to add the duration to a date.

```javascript
// Add the duration to the current time
const fromNow = value.toDate();

// Add the duration to the specific date and time
const fromSpecific = value.toDate(new Date(2015, 0, 2));
```

#### Example

```javascript
const { dateDuration } = require('ecolect/values');

builder.intent('listPending')
  .value('dateDuration', dateDuration())
  .add('Show todos due to be completed in {dateDuration}')
  .done();
```

### Time Duration

Capture a duration of hours, minutes, seconds and miliseconds.

Language         | Examples
-----------------|-------------
English          | `2 hours`, `1s`, `2h, 45m`, `4 minutes and 10 seconds`

#### Returned value

The returned value is an object containg fields with the change, such as
`hours`, `minutes`, `seconds` and `milliseconds`. The function 
`toDate(currentTime)` can be used to add the duration to a date.

```javascript
// Add the duration to the current time
const fromNow = value.toDate();

// Add the duration to the specific date and time
const fromSpecific = value.toDate(new Date(2015, 0, 2, 10, 0));
```

#### Example

```javascript
const { timeDuration } = require('ecolect/values');

builder.intent('timer')
  .value('timeDuration', timeDuration())
  .add('Set a timer for {timeDuration}')
  .done();
```

### Date & Time Duration

Capture a duration of both days, hours, minutes, seconds and miliseconds.

Language         | Examples
-----------------|-------------
English          | `2 hours`, `2 d 20 m`, `4 weeks and 10 minutes`

#### Returned value

The returned value is an object containg fields with the change, such as
`years`, `weeks`, `months` and `days`, `hours`, `minutes`, `seconds` and
`milliseconds`. The function `toDate(currentTime)` can be used to add the
duration to a date.


```javascript
// Add the duration to the current time
const fromNow = value.toDate();

// Add the duration to the specific date and time
const fromSpecific = value.toDate(new Date(2015, 0, 2, 10, 0));
```

#### Example

```javascript
const { dateTimeDuration } = require('ecolect/values');

builder.intent('timer')
  .value('dateTimeDuration', dateTimeDuration())
  .add('Set a timer for {dateTimeDuration}')
  .done();
```

### Enumeration

Capture one of the specified values. Used to specify one or more values that
should match.

```javascript
const { enumeration } = require('ecolect/values');

builder.intent('list')
  .value('type', enumeration([
    'Balloons',
    'Cookies',
    'Tasty Cake'
  ]))
  .add('Show me the last {type}')
  .done();
```

### Text

Text can be captured with the type `any`. You can use `any` for things such as
search queries, todo items and calendar events. Values of type `any` will
always try to capture as much as they can and will not validate the result.

```javascript
const { any } = require('ecolect/values');

builder.intent('echo')
  .value('text', any())
  .add('Echo {text}')
  .done();
```
