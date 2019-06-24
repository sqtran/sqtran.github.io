---
layout: post
title: JDV OData
---

## JDV OData

In Jboss Data virtualization, all deployed VDBs automatically get an OData API if the OData module has been deployed on the server. At least with JDV 6.3, the OData module deploys by default.

To view the VDB's OData data, you can call the following from any web browser (even curl).  
~~~
http://<servername>:8080/odata/<vdbname>/$metadata
~~~
Gotcha: There is a bug in the JDV OData implementation that occurs when you have a stored procedure that doesn't return a value. The XML response does not properly handle the empty response, which means it will err on the server side and not return any response. To workaround this, make sure your stored procedures return a dummy value.  This is a known bug, and will be fixed in a future release.

Here's an example of an OData stored procedure with parameters - they need to be http query parameters.
~~~
http://<servername>:8080/odata/<vdbname>/<viewname>.<storedprocedure>?ParamA=1&ParamB=2
~~~

This is equivalent to running the following command directly against the VDB with SQL.
```sql
call <vdbname>.<viewname>.<storedprocedure>(1,2);
```
The only difference is OData returns a lot of metadata in its response, which will need to be parsed by a client, or just ignored if not needed.

One way to get around the verbosity of XML is to use JSON format. Odata XML response can be converted to JSON but appending `$format=json` to the URL.
