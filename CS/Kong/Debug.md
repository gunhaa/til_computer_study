# Kong Debug


## The problem: high CPU usage in Kong servers

> https://blog.openresty.com/en/lua-str-lower-excep/
> 

- Our customer noticed that their Kong servers were consuming more CPU resources than expected, even though the incoming API traffic was not very high. They suspected that there might be some inefficiencies or errors in their custom plugins, but they had no clue where to look for them. They needed a tool that could help them pinpoint the root cause of the CPU bottleneck and provide actionable insights on how to fix it.
- The customer installed OpenResty XRay on their Kong servers and configured it to automatically sample the online Kong processes either periodically or when the CPU usage spiked. OpenResty XRay also automatically updated the analysis report for the current day every hour, so that our customer and our team could see the latest performance data.