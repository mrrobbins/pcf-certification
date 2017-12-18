# pcf-certification
Study material for Pivotal Cloud Foundry certification exams

# Diego Architecture

## Auctioning

https://docs.pivotal.io/pivotalcf/1-12/concepts/diego/diego-auction.html

- Balances process jobs over the available VM instances.
- Jobs can be one of two types:
  - Tasks (finite run time) e.g. a database initialization or service setup
  - Long-Running Processes (LRP) e.g. a running web application


### Auction priorities

1. Keep at least one instance of each LRP running.
2. Run all of the Tasks in the current batch.
3. Distribute as much of the total desired LRP load as possible over the remaining available VMs, by spreading multiple LRP instances broadly across VMs and their Availability Zones.

### Auction flow

1. Sort all Tasks and LRPs in order.
  - LPRs have a count and a size (number of instances and memory)
  - Round robin order each LPR, each round sorted by size.
  - If there are Tasks, insert them into the order after the first round.
  - Example final priority order:
  ```
    large-ram-LRP-1
    medium-ram-LRP-1
    small-ram-LRP-1
    task-a
    task-b
    large-ram-LRP-2
    medium-ram-LRP-2
    small-ram-LRP-2
    task-c
    small-ram-LRP-3
    small-ram-LRP-4
  ```
2. Auction the batch to cells (VMs) Each VM has a Cell Rep that participates in the auction on the VM's behalf

### Auction triggers

- BBS (Bulletin Board System) monitors desired LRPs instance count vs actually running and will trigger and auction if they are different
- Cloud Controller monitors cells for failure and will trigger an auction
- If a cell responds it cannot accommodate the workload, the unallocated work is carried over to the next Auction
- Cells just assigned a workload that become unresponsive does not automatically trigger a new auction. System defers to BBS to monitor counts on its next checking cycle.


# Availability Zones

- There are 4 levels of availability zones.
- An availability zone typically has a 1x1 mapping to physical rack in a datacenter

## BOSH Managed Processes

- Elastic runtime processes (e.g. Cloud Controller) are monitored and automatically restarted by `Monit`.
- Monit will report restarts to the `BOSH Agent` which will report it to `B osh`.
- BOSH is a VM responsible for managing the health of the entire system.
- BOSH sends heartbeats to the health monitor via the message bus
- humans can be notification
- What if entire VM is destroyed? Then `Health Monitor Resurrector` will request the BOSH Director to provision a new VM. This is an example of self-healing

# Application Scaling

## Vertical Scaling

Increasing the memory of existing app instances

For example:
`cf scale <app-name> -m 1GB`

- Container must be destroyed, causing application restart and downtime
- Downtime can be mitigated using the blue-green technique.
- Does this cause downtime if you have more than one instance?

## Horizontal Scaling

Increasing the number of instances of an app

`cf scale <app-name> -i 3`

- No downtime. Only spinning up new containers. Existing containers don't need to change.
