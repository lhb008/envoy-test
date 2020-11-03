Overload manager {#arch_overview_overload_manager}
================

The overload manager is an extensible component for protecting the Envoy
server from overload with respect to various system resources (such as
memory, cpu or file descriptors) due to too many client connections or
requests. This is distinct from
`circuit breaking <arch_overview_circuit_break>`{.interpreted-text
role="ref"} which is primarily aimed at protecting upstream services.

The overload manager is
`configured <config_overload_manager>`{.interpreted-text role="ref"} by
specifying a set of resources to monitor and a set of overload actions
that will be taken when some of those resources exceed certain pressure
thresholds.

Architecture
------------

The overload manager works by periodically polling the *pressure* of a
set of **resources**, feeding those through **triggers**, and taking
**actions** based on the triggers. The set of resource monitors,
triggers, and actions are specified at startup.

### Resources

A resource is a thing that can be monitored by the overload manager, and
whose *pressure* is represented by a real value in the range \[0, 1\].
The pressure of a resource is evaluated by a *resource monitor*. See the
`configuration page <config_overload_manager>`{.interpreted-text
role="ref"} for setting up resource monitors.

### Triggers

Triggers are evaluated on each resource pressure update, and convert a
resource pressure value into an action state. An action state has a
value in the range \[0, 1\], and is categorized into one of two groups:

::: {#arch_overview_overload_manager-triggers-state}
  -----------------------------------------------------------------------
  action state      value             description
  ----------------- ----------------- -----------------------------------
  scaling           \[0, 1)           the resource pressure is below the
                                      configured saturation point; action
                                      may be taken

  saturated         1                 the resource pressure is at or
                                      above the configured saturation
                                      point; drastic action should be
                                      taken
  -----------------------------------------------------------------------
:::

When a resource pressure value is updated, the relevant triggers are
reevaluated. For each action with at least one trigger, the resulting
action state is the maximum value over the configured triggers. What
effect the action state has depends on the action\'s configuration and
implementation.

### Actions

When a trigger changes state, the value is sent to registered actions,
which can then affect how connections and requests are processed. Each
action interprets the input states differently, and some may ignore the
*scaling* state altogether, taking effect only when *saturated*.
