# Actions

Actions are a script or operation that is generally triggered by [Metrics](Metrics.md), [Monitoring](Monitoring), by some external service, or interactively by a person.

The action itself is mostly a LUA script (which may even be generic which then references some other scripting service).

## External Service triggers

If activated, the OpenCluster system can have some API's exposed (either internally or externally).  The API can be used to trigger some specific functionality.

> For example, if an organisation has OpenCluster services in the cloud, some cloud functionality may interact with OpenCluster to trigger some functionality.

## Interactively

On the OpenCluster management console (which can be customised to the organisations needs), they may have a control which starts some functionality.

> For example, in the management console, they may have some functionality which is hidden when everything is running good, but when an issue is detected, it makes the section visible, with actions presented that can be triggered.  Like it might display some buttons that cause everything to migrate to the cloud (if on-prem is unreliable).  Or if some customer-impacted issue is detected, it could display a section for staff to send out a notice to clients (with the ability to adjust the commentary).  This is all customised and very flexible.
