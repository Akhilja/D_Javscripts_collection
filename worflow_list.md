## get the list of worflows 

```java

import { workflowsClient } from "@dynatrace-sdk/client-automation";

export default async function () {
  const allWorkflows = [];
  const PAGE_SIZE = 100;
  let offset = 0;
  let totalCount = null;

  do {
    const response = await workflowsClient.getWorkflows({
      adminAccess: true,
      limit: PAGE_SIZE,
      offset: offset,
    });

    if (totalCount === null) {
      totalCount = response.count;
    }

    for (const wf of response.results ?? []) {
      let triggerStatus = "No trigger";
      const trigger = wf.trigger;

      if (trigger?.schedule) {
        triggerStatus = trigger.schedule.isActive
          ? "Schedule trigger (enabled)"
          : "Schedule trigger (disabled)";
      } else if (trigger?.eventTrigger) {
        triggerStatus = "Event trigger";
      } else if (trigger?.webhook) {
        triggerStatus = "Webhook trigger";
      }

      allWorkflows.push({
        id:            wf.id,
        name:          wf.title,
        state:         wf.isPrivate ? "draft (private)" : "live (public)",
        actor:         wf.actor ?? "N/A",
        owner:         wf.owner ?? wf.ownerId ?? wf.ownerName ?? "N/A",
        triggerStatus: triggerStatus,
      });
    }

    offset += PAGE_SIZE;

  } while (totalCount === null || offset < totalCount);

  console.table(allWorkflows);

  return allWorkflows;
}

```

## other option 

```java

import { workflowsClient } from "@dynatrace-sdk/client-automation";

export default async function () {
  const allWorkflows = [];
  const PAGE_SIZE = 100;
  let offset = 0;
  let totalCount = null;

  do {
    const response = await workflowsClient.getWorkflows({
      adminAccess: true,
      limit: PAGE_SIZE,
      offset: offset,
    });

    if (totalCount === null) {
      totalCount = response.count;
    }

    for (const wf of response.results ?? []) {
      let triggerStatus = "No trigger";
      const trigger = wf.trigger;

      if (trigger?.schedule) {
        triggerStatus = trigger.schedule.isActive
          ? "Schedule trigger (enabled)"
          : "Schedule trigger (disabled)";
      } else if (trigger?.eventTrigger) {
        triggerStatus = "Event trigger";
      } else if (trigger?.webhook) {
        triggerStatus = "Webhook trigger";
      }

      allWorkflows.push({
        id:            wf.id,
        name:          wf.title,
        state:         wf.isPrivate ? "draft (private)" : "live (public)",
        actor:         wf.actor ?? "N/A",
        owner:         wf.owner ?? wf.ownerId ?? wf.ownerName ?? "N/A",
        triggerStatus: triggerStatus,
      });
    }

    offset += PAGE_SIZE;

  } while (totalCount === null || offset < totalCount);

  // Print in chunks of 100 to bypass console.table display limit
  const CHUNK_SIZE = 100;
  for (let i = 0; i < allWorkflows.length; i += CHUNK_SIZE) {
    const chunk = allWorkflows.slice(i, i + CHUNK_SIZE);
    console.log(`--- Workflows ${i + 1} to ${Math.min(i + CHUNK_SIZE, allWorkflows.length)} ---`);
    console.table(chunk);
  }

  console.log(`Total workflows fetched: ${allWorkflows.length} / ${totalCount}`);

  return allWorkflows;
}

```
