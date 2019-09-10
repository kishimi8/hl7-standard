<h1 align="center">
  <br>
  <a href="https://ironbridgecorp.com"><img src="https://hl7-standard-images.s3.amazonaws.com/hl7-standard-logo-stamp.svg" alt="HL7 Standard" width="300"></a>
  <br>
  hl7-standard
  <br>
</h1>

*A simple, lightweight HL7 module for tranforming, manipulating, or creating HL7 messages*

## Description & Features

HL7-Standard is a javascript based library that aims to make handling HL7 data simpler.  This lightweight library was written and open sourced by [Iron Bridge](https://www.ironbridgecorp.com/), a healthcare technology company. HL7-Standard is able to be used as a stand-alone js scripting module or dropped into an application like mirth to aid in difficult transformations.

- [Mirth Compatibility](#mirth-usage)

---

## Usage

### Install

To install 'hl7-standard' for use in node via `require('hl7-standard')`, run:

```bash
npm install hl7-standard
```

## API Methods

HL7-Standard enables users to quickly manipulate HL7 data using JSON. It consists of the following methods:

- [transform](#transform)
- [build](#build)
- [get](#get)
- [getSegment](#getsegment)
- [getSegments](#getsegments)
- [getSegmentsAfter](#getsegmentsafter)
- [set](#set)
- [createSegment](#createsegment)
- [createSegmentAfter](#createsegmentafter)
- [createSegmentBefore](#createsegmentbefore)
- [deleteSegment](#deletesegment)
- [deleteSegments](#deletesegments)
- [moveSegmentAfter](#movesegmentafter)
- [moveSegmentBefore](#movesegmentbefore)

### transform

Transforms raw HL7 data to 'hl7-standard' constructor format for message manipulation

```js

let hl7 = new HL7(rawData);
hl7.transform(err => {
  if (err) throw err;
  // code here
});

// callback optional
try {
 let hl7 = new HL7(rawData);
 hl7.transform();
 // code here
} catch (e) {
  console.error(e);
}

```

### build

Builds a new HL7 message from transformed data

```js

const finalizedHL7 = hl7.build(); // MSH|^~\&|Example|General Hospital|ADT||20070510092626|411|ADT^A01|24553|P|2.4|||....
fs.writeFileSync(`${__dirname}/data/hl7_message_${uuid}.hl7`, finalizedHL7, 'utf8');

```

### get

Gets a value from a transformed segment returning an array, object, or string

```js

let familyName = hl7.get('PID.5.1'); //Reynolds

let patientLanguage = hl7.get('PID.15'); // { 'PID.15.1':'en', 'PID.15.2':'English' }

```

### getSegment

Gets a single segment constructor, if multiple exist, it will return the first one

```js

let pidSegment = hl7.getSegment('PID');
pidSegment.get('PID.3.1'); //123456

```

### getSegments

Gets all segment constructors by segment type, if no segment is specified, method will return all segments in the message

```js
const children = hl7.getSegments();
for (let i = 0; i < children.length; i++) {
  if (children[i].type === 'NTE') {
    // loop through every segment in the HL7 message and check for segment type
  }
}

for (let nte of hl7.getSegment('NTE')) {
  // loop through only the NTE segments using getSegments
}

```

### getSegmentsAfter

Returns all requested 'Segment' constructors at occur in the HL7 message after a specified start point

```js

for (let obr of hl7.getSegment('OBR')) {
  if (obr.get('OBR.4.1') === '1234x') {
    let nteGroup = hl7.getSegmentsAfter(obr, 'NTE', false, ['ORC', 'OBR']);
    // nteGroup contains all NTE segments after the OBR segment and before the next OBR or ORC segment
  }
}

```

### set

Sets a value on a transformed segment

```js

hl7.set('PID.5.1', 'Reynolds');

hl7.set('PID.15', { 'PID.15.1':'en', 'PID.15.2':'English' });

hl7.set('PID.11', [{
  'PID.11.1': '123 Example Rd',
  'PID.11.2': '',
  'PID.11.3': 'Pittsburgh',
  'PID.11.4': 'PA',
  'PID.11.4': '15226',
  'PID.11.5': 'USA'
}, {
  'PID.11.1': '321 Sample St',
  'PID.11.2': '',
  'PID.11.3': 'Pittsburgh',
  'PID.11.4': 'PA',
  'PID.11.4': '15317',
  'PID.11.5': 'USA'
}]);

```

### createSegment

Creates a single segment at the end of the hl7 message

```js

let hl7 = new HL7();
hl7.createSegment('MSH');
hl7.set('MSH', {
  'MSH.2': '^~\\&',
  'MSH.3': 'Example',
  'MSH.4': '123456',
  'MSH.5': '',
  'MSH.6': '',
  'MSH.7': timestamp,
  'MSH.8': '',
  'MSH.9': {
    'MSH.9.1': 'ADT',
    'MSH.9.2': 'A08'
  },
  'MSH.10': '',
  'MSH.11': 'T',
  'MSH.12': '2.3'
});

```

### createSegmentAfter

Creates a single segment after an already existing, specified segment

```js

let pv1Segment = hl7.createSegmentAfter('PV1', hl7.getSegment('PID'));
pv1Segment.set(...);

```

### createSegmentBefore

Creates a single segment before an already existing, specified segment

```js

let evnSegment = hl7.createSegmentAfter('EVN', hl7.getSegment('PID'));
evnSegment.set(...);

```

### deleteSegment

Deletes a single segment from the HL7 message

```js

for (let segment of hl7.getSegment()) {
  if (segment.type === 'ZIB') hl7.deleteSegment(segment);
}

```

### deleteSegments

Deletes multiple segments from the HL7 message

```js

// conditionally delete a block of NTE segments from a message
for (let obr of hl7.getSegments('OBR')) {
  if (obr.get('OBR.4.1') === '681X') {
    let nteGroup = hl7.getSegmentsAfter(obr, 'NTE', true, 'OBR');
    hl7.deleteSegments(nteGroup);
  }
}

//delete all NTE segments from message
hl7.deleteSegments(hl7.getSegments('NTE'))
  
```

### moveSegmentAfter

Moves existing segment after another existing segment

```js

let hl7 = new HL7();
let evnSegment = hl7.createSegment('EVN');
evnSegment.set('EVN.1.1', 'A08');
hl7.moveSegmentAfter(evnSegment, hl7.getSegment('MSH'));

```

### moveSegmentBefore

Moves existing segment before another existing segment

```js

let hl7 = new HL7();
let evnSegment = hl7.createSegment('EVN');
evnSegment.set('EVN.1.1', 'A08');
hl7.moveSegmentBefore(evnSegment, hl7.getSegment('PID'));

```
---

## Mirth Usage

Adding 'hl7-standard' to Mirth opens up more possibilities for data tranformations. This 20KB library can make some of the advanced mirth transformations much simplier and can aid in readability and ease of use. Can you get by in Mirth without using 'hl7-standard', absolutely; however, we find this library to be very intuitive and easier for a novice to pick up and run with.

### Configuring 'hl7-standard' in Mirth

To use the library in mirth, you will need to copy the `mirth.min.js` file out of the `builds/` directory and import it into your mirth code templates. On newer versions of mirth make sure you give the template the context of source connector and destination connector and make sure the 'Library' that houses the code template is set to add the lib to any new channel. Alternatively, you can control what channels have access to the library individually using the code template options.

### Setting the Data Types to Raw on a Mirth Channel

There are many ways to incorporate 'hl7-standard' into your channels workflow, but the simplest method would be to set the source data type to `raw` allowing the raw `msg` to be a string.  Channel inbound and outbound data type settings can be found under the `Summary` tab of your Mirth channel.

<img src="https://hl7-standard-images.s3.amazonaws.com/channel-datatypes.png" alt="HL7 Standard" width="600">

### Setting the Dependencies on a Channel

If you set the code template options to auto-set the channel dependencies you can skip this step, but you can always check the channel dependencies to ensure the library is accessible by your channel. Channel dependencies can be found under the `Summary` tab of your Mirth channel.

<img src="https://hl7-standard-images.s3.amazonaws.com/channel-dependencies.png" alt="HL7 Standard" width="600">

### Example Mirth Transformer Using 'hl7-standard'

There are many ways to transform HL7 data with 'hl7-standard'. Below is a sample destination transformer with a HL7 transformation that involves conditionally deleting unwanted NTE segments. More examples like this can be found in the `examples/` directory and on the Iron Bridge Developer Site.

<img src="https://hl7-standard-images.s3.amazonaws.com/example-transformation.png" alt="HL7 Standard" width="600">

### Example Preprocessor Using 'hl7-standard'

A common use case for utilizing 'hl7-standard' in mirth outside of channel transformations is evaluating an incoming message within the preprocessor. Sometimes, interface feeds receive unwanted messages that are not HL7 and they tend to cause errors and issues in the interface workflow. By adding 'hl7-standard', we can evaluate the message in the pre-processor, prior to the message being parsed by mirth.

<img src="https://hl7-standard-images.s3.amazonaws.com/preprocessor-flagging.png" alt="HL7 Standard" width="600">


