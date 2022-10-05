# how we add geocoding 
We use nomatim API for OpenStreetMap at https://nominatim.openstreetmap.org/search to find longitude/latitude and bounding box for publication affilations.

Tables for this are <tt>affiliation_geo</tt> and <tt>publication_affiliation_geo</tt>.
Former holds data retrieved form openstrreetmap for a query string. Only unique query strings are queries, i.e. we do not store affiliations per run.<br/>
Latter links the publication_affiliation table to the affiliation_geo table.

\NB the algorithm also requires a table linking the 3=letter country code to the country name used in the request.

Algorithm:
* derive from publication_affiliation table a comma-separated query string composed of<br/>
  (institution_name,address,city,state,country name): <tt>q</tt><br/>
  (country name is derived from the three-letter country_code)<br/)
  and check in affiliation_geo table whether a result has already been found for <tt>q</tt><br/>    
  * <b>SQL</b>:<br/><br/>
```sql
with pa as (
select distinct concat_ws(',',pa.institution_name,replace(pa.address,'|',','),pa.city,pa.state,cc.country_name) as q
,      pa.institution_name,pa.address, pa.city, pa.state,pa.country_code
  from publication_affiliation pa
  left outer join country_code cc on cc.country_code=pa.country_code
)
select pa.*
  from pa
 where not exists (
   select q_final 
     from affiliation_geo ag 
    where ag.q=pa.q
      and ag.q_final is not null
   )
```
<br/>
* the <tt>q</tt> values from the result of this query are submitted to nomatim in an iterative manner: 
  * if no result is returned: cut left-most substring before comma and that comma, resubmit.<br/>
    keep counter nattempts
  * if a result is returned, generally >1 results, append to pandas dataframe: <br/>
    (q,q_final,nattempts,lon,lat,boundingbox,display_name,importance) <br/>
    write to CSV file
* load CSV file to <tt>affiliation_geo_staging</tt> table in ShowUStheData_staging database
* from there copy data to <tt>affiliation_geo</tt> table using following query
  * <b>SQL</b>:<br/><br/>
```sql

```
  * the IDENTIY <tt>id</tt> column wil be set automatically
  * create SQL Server geography columnes: geo_location (Point from lat/lon), geo_boundingbox (POLYGON from boundingbox)
* load data intopublication_affiliation_geo table using following script
```sql

```

