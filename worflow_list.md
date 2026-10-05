## get the worflow list 

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

    // On first page, log the raw first workflow object to inspect all available fields
    if (offset === 0 && response.results?.length > 0) {
      console.log("Raw workflow object (first item):", JSON.stringify(response.results[0], null, 2));
      totalCount = response.count; // total number of workflows available
      console.log(`Total workflows available: ${totalCount}`);
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
        // Try all known owner field variants — check the raw log above to confirm the right one
        owner:         wf.owner ?? wf.ownerId ?? wf.ownerName ?? "N/A",
        triggerStatus: triggerStatus,
      });
    }

    offset += PAGE_SIZE;

  } while (totalCount === null || offset < totalCount);

  console.log(`Total workflows fetched: ${allWorkflows.length}`);
  console.table(allWorkflows);

  return allWorkflows;
}

```
