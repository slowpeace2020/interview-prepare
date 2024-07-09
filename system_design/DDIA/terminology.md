# Terminology

## How to measure response time? How it affect business?

If you take your list of response times and sort it
from  fastest  to  slowest,  then  the  median  is  the  halfway  point:  for  example,  if  your median  response  time  is  200  ms,  that  means  half  your  requests  return  in  less  than
200 ms, and half your requests take longer than that.  
This  makes  the  median  a  good  metric  if  you  want  to  know  how  long  users  typically
have  to  wait:  half  of  user  requests  are  served  in  less  than  the  median  response  time,
and the other half take longer than the median.   
The median is also known as the 50th
percentile, and sometimes abbreviated as p50. Note that the median refers to a single
request;  if  the  user  makes  several  requests  (over  the  course  of  a  session,  or  because
several  resources  are  included  in  a  single  page),  the  probability  that  at  least  one  of
them is slower than the median is much greater than 50%.

In  order  to  figure  out  how  bad  your  outliers  are,  you  can  look  at  higher  percentiles:
the 95th, 99th, and 99.9th percentiles are common (abbreviated p95, p99, and p999).
They  are  the  response  time  thresholds  at  which  95%,  99%,  or  99.9%  of  requests  are
faster than that particular threshold. For example, if the 95th percentile response time
is 1.5 seconds, that means 95 out of 100 requests take less than 1.5 seconds, and 5 out
of 100 requests take 1.5 seconds or more. 

High percentiles of response times, also known as tail  latencies, are important
because  they  directly  affect  users’  experience  of  the  service.

Amazon has also observed that a 100 ms increase in response time reduces
sales by 1% , and others report that a 1-second slowdown reduces a customer sat‐
isfaction metric by 16%.

or  example,  percentiles  are  often  used  in  service  level  objectives  (SLOs)  and  service
level  agreements  (SLAs),  contracts  that  define  the  expected  performance  and  availa‐
bility of a service. An SLA may state that the service is considered to be up if it has a
median  response  time  of  less  than  200  ms  and  a  99th  percentile  under  1  s  (if  the
response time is longer, it might as well be down), and the service may be required to
be up at least 99.9% of the time. These metrics set expectations for clients of the ser‐
vice and allow customers to demand a refund if the SLA is not met.


