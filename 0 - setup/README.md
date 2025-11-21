# Setup

![Pentesting Diagram](../../assets/pentestingprocess.png)

`Black box` pentesting is done with no knowledge of a network's configuration or applications. Typically a tester will either be given network access (or an ethernet port and have to bypass Network Access Control NAC) and nothing else (requiring them to perform their own discovery for IP addresses) if the pentest is internal, or nothing more than the company name if the pentest is from an external standpoint. This type of pentesting is usually conducted by third parties from the perspective of an `external` attacker. Often the customer will ask the pentester to show them discovered internal/external IP addresses/network ranges so they can confirm ownership and note down any hosts that should be considered out-of-scope.

`Grey box` pentesting is done with a little bit of knowledge of the network they're testing, from a perspective equivalent to an `employee` who doesn't work in the IT department, such as a `receptionist` or `customer service agent`. The customer will typically give the tester in-scope network ranges or individual IP addresses in a grey box situation.

`White box` pentesting is typically conducted by giving the penetration tester full access to all systems, configurations, build documents, etc., and source code if web applications are in-scope. The goal here is to discover as many flaws as possible that would be difficult or impossible to discover blindly in a reasonable amount of time.