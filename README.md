# InfoHub-Profiles

Profile definitions for publishing industrial data using a composable, standardised payload structure. These profiles are designed as reusable building blocks that can be composed into complete payload schemas for different data types.

Built for use with [Flow Software](https://www.flowsoftware.com) and the [CESMII Smart Manufacturing Innovation Institute](https://www.cesmii.org).

## Overview

These JSON Schema profiles define the payload structure for publishing measure and event data from Flow Software to an MQTT broker using the **SM Profile** protocol. The profiles follow a composable "lego block" architecture — each sub-section is an independently referenceable building block that can be assembled into complete payload schemas.

Each building block includes:
- **`$comment`** — Human-readable description of the block's purpose
- **`$namespace`** — Profile namespace URI linking to the published SM Profile (e.g. `{profileUri}/Measure`)

## Profile Architecture

```
MeasureValueData                    EventSchemeData                 MeasureEventPeriodValueData
  +-- Measure                         +-- EventScheme                 +-- Measure
  +-- TimePeriod                      +-- EventPeriod                 +-- EventScheme
  +-- MeasureValue[]                  +-- EventAttributeValue[]       +-- EventPeriod
  +-- ModelAttributes (optional)      +-- ModelAttributes (optional)  +-- MeasureEventPeriodValue[]
                                                                      +-- ModelAttributes (optional)
```

## Building Block Schemas

Individual reusable components that can be referenced via `$ref`:

| Schema | Description |
|--------|-------------|
| [FlowMeasure.schema.json](FlowMeasure.schema.json) | Measure identification — name, ID, UOM, interval type, API endpoints |
| [FlowTimePeriod.schema.json](FlowTimePeriod.schema.json) | Reporting time window — ISO 8601 and Unix timestamps, time scheme |
| [FlowMeasureValue.schema.json](FlowMeasureValue.schema.json) | Individual data point — value, quality, duration, optional attribute breakdown |
| [FlowModelAttributes.schema.json](FlowModelAttributes.schema.json) | Contextual metadata from the data model hierarchy (dynamic key-value pairs) |

### Composite Schema (with $ref)

| Schema | Description |
|--------|-------------|
| [FlowMeasureValueData.schema.json](FlowMeasureValueData.schema.json) | Root composite — references building blocks via `$ref` |

## Combined Schemas (Self-Contained)

Complete, self-contained schemas with all definitions inlined. Use these when you need a single file without external references — ideal for uploading to the [CESMII Profile Designer](https://profiledesigner.cesmii.net) or for systems that don't support `$ref`.

| Schema | Description |
|--------|-------------|
| [FlowMeasureValueData.combined.schema.json](FlowMeasureValueData.combined.schema.json) | Measure value data — one message per time period with aggregated values |
| [FlowEventSchemeData.combined.schema.json](FlowEventSchemeData.combined.schema.json) | Event scheme data — one message per event period with attribute values |
| [FlowMeasureEventPeriodValueData.combined.schema.json](FlowMeasureEventPeriodValueData.combined.schema.json) | Measure event period value data — measure values within event periods |

## Payload Examples

### Measure Value Data

Published per time period with all measure values aggregated into a single message.

```json
{
  "$comment": "Flow Software CESMII SM Profile - Measure Value Data",
  "$namespace": "https://profiledesigner.cesmii.net/schemas/my-profile/MeasureValueData",
  "measure": {
    "$comment": "Measure definition — identifies the measure being reported",
    "$namespace": "https://profiledesigner.cesmii.net/schemas/my-profile/Measure",
    "name": "Electricity Consumption",
    "id": 1042,
    "uom": "kWh",
    "description": "Hourly electricity consumption",
    "intervalType": "Hourly",
    "hierarchicalName": "Plant1.Line2.Electricity Consumption",
    "parent": "Line2",
    "format": "0.00",
    "measureDataApiEndpoint": "https://host/flow-software/.../api/v1/data/measures?measureId=1042",
    "measureDataApiEndpointWithAttributes": "https://host/flow-software/.../api/v1/data/measures?measureId=1042&attributes=true"
  },
  "timePeriod": {
    "$comment": "Time period — defines the reporting window for the values",
    "$namespace": "https://profiledesigner.cesmii.net/schemas/my-profile/TimePeriod",
    "start": "2024-01-15T08:00:00.0000000Z",
    "end": "2024-01-15T09:00:00.0000000Z",
    "startUnix": 1705305600,
    "endUnix": 1705309200,
    "timeScheme": "Calendar",
    "timeSchemeShift": null
  },
  "values": [
    {
      "$comment": "Measure value — a single data point with optional attribute breakdown",
      "$namespace": "https://profiledesigner.cesmii.net/schemas/my-profile/MeasureValue",
      "value": 42.5,
      "quality": 192,
      "formattedValue": "42.50",
      "duration": 3600
    },
    {
      "$comment": "Measure value — a single data point with optional attribute breakdown",
      "$namespace": "https://profiledesigner.cesmii.net/schemas/my-profile/MeasureValue",
      "value": 38.2,
      "quality": 192,
      "formattedValue": "38.20",
      "duration": 3600,
      "attribute": "Phase",
      "attributeValue": "L1"
    }
  ],
  "modelAttributes": {
    "$comment": "Model attributes — contextual metadata from the data model hierarchy",
    "$namespace": "https://profiledesigner.cesmii.net/schemas/my-profile/ModelAttributes",
    "Site": "Plant1",
    "Area": "Line2"
  }
}
```

### Event Scheme Data

Published per event period with all event attribute values aggregated.

```json
{
  "$comment": "Flow Software CESMII SM Profile - Event Scheme Data",
  "$namespace": "https://profiledesigner.cesmii.net/schemas/my-profile/EventSchemeData",
  "eventScheme": {
    "$comment": "Event scheme definition",
    "$namespace": "https://profiledesigner.cesmii.net/schemas/my-profile/EventScheme",
    "id": 5,
    "name": "Production Run",
    "description": "Tracks production run periods"
  },
  "eventPeriod": {
    "$comment": "Event period — defines the event time window",
    "$namespace": "https://profiledesigner.cesmii.net/schemas/my-profile/EventPeriod",
    "start": "2024-01-15T06:00:00.0000000Z",
    "end": "2024-01-15T14:00:00.0000000Z",
    "startUnix": 1705298400,
    "endUnix": 1705327200,
    "startQuality": 192,
    "endQuality": 192
  },
  "attributeValues": [
    {
      "$comment": "Event attribute value",
      "$namespace": "https://profiledesigner.cesmii.net/schemas/my-profile/EventAttributeValue",
      "attribute": "Product",
      "value": "Widget-A"
    },
    {
      "$comment": "Event attribute value",
      "$namespace": "https://profiledesigner.cesmii.net/schemas/my-profile/EventAttributeValue",
      "attribute": "Batch",
      "value": "B-2024-0115"
    }
  ],
  "modelAttributes": {
    "$comment": "Model attributes — contextual metadata from the data model hierarchy",
    "$namespace": "https://profiledesigner.cesmii.net/schemas/my-profile/ModelAttributes",
    "Site": "Plant1",
    "Area": "Line2"
  }
}
```

### Measure Event Period Value Data

Published per event period with measure values within the event context.

```json
{
  "$comment": "Flow Software CESMII SM Profile - Measure Event Period Value Data",
  "$namespace": "https://profiledesigner.cesmii.net/schemas/my-profile/MeasureEventPeriodValueData",
  "measure": {
    "$comment": "Measure definition — identifies the measure being reported",
    "$namespace": "https://profiledesigner.cesmii.net/schemas/my-profile/Measure",
    "name": "Electricity Consumption",
    "id": 1042,
    "uom": "kWh",
    "description": "Hourly electricity consumption",
    "intervalType": "Hourly",
    "hierarchicalName": "Plant1.Line2.Electricity Consumption",
    "parent": "Line2",
    "format": "0.00",
    "measureDataApiEndpoint": "https://host/flow-software/.../api/v1/data/measures?measureId=1042&eventId=5",
    "measureDataApiEndpointWithAttributes": "https://host/flow-software/.../api/v1/data/measures?measureId=1042&eventId=5&attributes=true"
  },
  "eventScheme": {
    "$comment": "Event scheme definition",
    "$namespace": "https://profiledesigner.cesmii.net/schemas/my-profile/EventScheme",
    "id": 5,
    "name": "Production Run"
  },
  "eventPeriod": {
    "$comment": "Event period — defines the event time window",
    "$namespace": "https://profiledesigner.cesmii.net/schemas/my-profile/EventPeriod",
    "start": "2024-01-15T06:00:00.0000000Z",
    "end": "2024-01-15T14:00:00.0000000Z",
    "startUnix": 1705298400,
    "endUnix": 1705327200
  },
  "values": [
    {
      "$comment": "Measure event period value",
      "$namespace": "https://profiledesigner.cesmii.net/schemas/my-profile/MeasureEventPeriodValue",
      "value": 342.8,
      "quality": 192,
      "eventAttributes": {
        "Product": "Widget-A",
        "Batch": "B-2024-0115"
      }
    }
  ],
  "modelAttributes": {
    "$comment": "Model attributes — contextual metadata from the data model hierarchy",
    "$namespace": "https://profiledesigner.cesmii.net/schemas/my-profile/ModelAttributes",
    "Site": "Plant1",
    "Area": "Line2"
  }
}
```

## Namespace Convention

Every section uses a `$namespace` property pointing to the published profile URI:

| Namespace Suffix | Block |
|-----------------|-------|
| `/MeasureValueData` | Root measure value payload |
| `/EventSchemeData` | Root event scheme payload |
| `/MeasureEventPeriodValueData` | Root measure event period value payload |
| `/Measure` | Measure identification |
| `/TimePeriod` | Reporting time window |
| `/MeasureValue` | Individual measure data point |
| `/MeasureEventPeriodValue` | Measure value within an event period |
| `/EventScheme` | Event scheme identification |
| `/EventPeriod` | Event time window |
| `/EventAttributeValue` | Event attribute value |
| `/ModelAttributes` | Model hierarchy metadata |

The base URI is configured in Flow Software under the consumer's **Profile URI** property (e.g. `https://profiledesigner.cesmii.net/schemas/my-profile`).

## Key Design Decisions

- **One message per period** — Each MQTT message contains all values for a single time/event period, aggregated into a `values` or `attributeValues` array
- **Composable building blocks** — Each sub-section is an independently referenceable profile with its own `$comment` and `$namespace`
- **Conditional sections** — `modelAttributes` is only included when model attributes exist; `attribute`/`attributeValue` on measure values only appear when event attributes are present
- **OPC quality codes** — The `quality` field uses standard OPC quality codes (e.g. 192 = Good)
- **Dual timestamp formats** — Both ISO 8601 and Unix timestamps are provided for maximum interoperability
- **API endpoint references** — Each measure includes direct API endpoint URLs for retrieving the source data, with and without attribute breakdowns

## JSON Schema Version

All schemas use [JSON Schema Draft 2020-12](https://json-schema.org/draft/2020-12/schema).

## License

Copyright Flow Software. All rights reserved.

