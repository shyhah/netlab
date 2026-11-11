# Week 4 lab: OpenFlow by hand

Network: one OpenFlow switch (s1) and three hosts: h1 (10.40.0.1), h2 (10.40.0.2) and h3 (10.40.0.3).

## 1. Why did the first firewall rule in Step 3 have no effect, and why does blocking h1 → h3 also break h3 → h1?

The first drop rule had the same priority as the rule that forwards traffic to h3. When two rules have the same priority, OpenFlow does not say which one wins, so the switch used the forwarding rule and the ping still worked. Giving the drop rule a higher priority (40000) made it win.

h3 → h1 also breaks because a ping needs a reply. h1's reply goes from 10.40.0.1 to 10.40.0.3, which matches the drop rule, so the reply is dropped.

## 2. In Step 5, why did one conversation survive the controller failure and another didn't?

h1 → h2 had already been set up by the controller, so its flow rules were still in the switch. h1 → h3 was new, so the switch had to ask the controller, but the controller was gone. After 60 seconds, the h1 → h2 rules expired, so h1 → h2 stopped working too.

## 3. Secure or standalone: which would you choose for a campus network, and what does the other one cost you?

I would choose secure, because the switch keeps following the firewall rules even when the controller fails. The cost is that new traffic stops until the controller comes back.

Standalone keeps the network working when the controller fails, because the switch acts like a normal learning switch. The cost is that it ignores the firewall rules, so blocked traffic can get through.
