
# Direct Object Rights

This is a cypher query **used in bloodhound**. See how to set up bloodhound at [8. Lateral Movement/AD Enumeration/Bloodhound setup#Server]({{< relref "8. Lateral Movement/AD Enumeration/Bloodhound setup#server" >}})

This query shows you what rights accounts have **directly assigned**. Very good in CTF or small AD environments.

```cypher
MATCH p=(source)-[r]->(target)
WHERE (source:Computer OR source:User)
AND type(r) <> 'MemberOf'
return p
```