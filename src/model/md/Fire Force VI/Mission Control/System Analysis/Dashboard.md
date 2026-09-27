---
ontology: https://fireforce6.github.io/mission-control/bundle
---

# System Analysis Dashboard

Dashboard for monitoring subsystem interfaces, physical connectivity, interface coverage, and method rule conformance.

---

## 1. Interface Integration and Integrity Assessment
<!-- NBPM: Turn one query into a dashboard section: state the engineering question, show the query result as evidence, and explain what the result means. Question -> Evidence -> Interpretation -->

### Engineering Question
> **Are all subsystems properly integrated into the command, telemetry, and power distribution topologies, or do orphan interfaces exist that leave flight hardware uncommanded or unmonitored?**

### Evidence

#### A. Near-Miss Interface Detection
Identifies subsystems that have declared ports and at least one active connection, but still have unconnected interfaces:

```compose
template: https://www.modelware.io/sierra/system-analysis/near-misses
```

#### B. Orphan Interface Detection
Identifies specific ports that have zero incoming or outgoing connections:

```compose
template: https://www.modelware.io/sierra/system-analysis/orphans
```

### Interpretation

Subsystems appearing in the near-miss detection (such as in our case for this assignment submission, `FireSat` and `PropulsionSegment`) reveal partial interface integration. The systems engineers have presumable clearly established their initial links (such as structural mounting or power distribution), but cross-discipline interface routing remains incomplete. In flight projects, this typically points to integration debt or handoff disconnects between subsystem teams where one engineering domain finished its routing while command, telemetry, or data handling paths were deferred.

At the port level, orphan interfaces point to at least two risks. Unconnected input ports represent operational vulnerabilities: for example, if the `PropulsionSegment` has its electrical power interface connected (`Platform.Power_Out` to `Propulsion.Power_In`), but its command receiver port `Propulsion.Command_In` remains unconnected, thruster valve actuation and burn timing will not function during flight operations because the flight computer has no physical command path to the hardware. Conversely, unconnected output ports result in telemetry blind spots where onboard sensors generate data that no downstream subsystem ever consumes or downlink records. In some cases, orphans simply represent model drift (interfaces drafted during early conceptual stages that were later determined unneeded but never removed) which distorts interface counts and clutters the system interface specs until eventually cleaned up. 

---

## 2. Interface "Health"
<!-- NBPM: Scripted block: query > compute > render -->

```python
include('src/method/py/utils.py')
include('src/method/py/oml_adapter.py')

# pass async query function into adapter
adapter = OmlQueryAdapter(query_executor=query)

# run canonical .sparql query from file via adapter
result = await adapter.execute('ports_health')

# Compute: calc interface completeness metrics
rows = result.get('rows', [])
total_ports = len(rows)
connected_ports = sum(1 for r in rows if str(r.get('isConnected', '')).lower() == 'true')
orphan_ports = total_ports - connected_ports
completeness_pct = round((connected_ports / total_ports * 100), 1) if total_ports > 0 else 0
health_color = "#27ae60" if completeness_pct >= 90 else "#f39c12" if completeness_pct >= 70 else "#e74c3c"

# Render: cards
display(f"""
<div style="display:flex; gap:16px; margin: 16px 0; flex-wrap:wrap;">
  <div style="background:#fdfefe; border:1px solid #e1e8ed; border-left:5px solid {health_color}; border-radius:6px; padding:14px 20px; flex:1; min-width:180px;">
    <div style="font-size:11px; text-transform:uppercase; color:#7f8c8d; font-weight:600;">Interface Completeness</div>
    <div style="font-size:28px; font-weight:bold; color:{health_color}; margin-top:4px;">{completeness_pct}%</div>
    <div style="font-size:12px; color:#95a5a6; margin-top:2px;">{connected_ports} of {total_ports} ports routed</div>
  </div>
  <div style="background:#fdfefe; border:1px solid #e1e8ed; border-left:5px solid #e74c3c; border-radius:6px; padding:14px 20px; flex:1; min-width:180px;">
    <div style="font-size:11px; text-transform:uppercase; color:#7f8c8d; font-weight:600;">Orphan Ports</div>
    <div style="font-size:28px; font-weight:bold; color:#e74c3c; margin-top:4px;">{orphan_ports}</div>
    <div style="font-size:12px; color:#95a5a6; margin-top:2px;">Unrouted interfaces</div>
  </div>
  <div style="background:#fdfefe; border:1px solid #e1e8ed; border-left:5px solid #2980b9; border-radius:6px; padding:14px 20px; flex:1; min-width:180px;">
    <div style="font-size:11px; text-transform:uppercase; color:#7f8c8d; font-weight:600;">Active Connections</div>
    <div style="font-size:28px; font-weight:bold; color:#2980b9; margin-top:4px;">{connected_ports // 2}</div>
    <div style="font-size:12px; color:#95a5a6; margin-top:2px;">Verified links</div>
  </div>
</div>
""")
```

---

## 3. Interface Connectivity Analysis

Let's render this section from a compose template:

```compose
# NBPM: "Package one analysis or view as a reusable compose template" -- look ma, we did it again!
template: https://www.modelware.io/sierra/system-analysis/interface-analysis
```
