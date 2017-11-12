

{
    "authors": [
        {
            "name": "Eric Busboom"
        }
    ],
    "date": "2017-11-11T21:00:50.207275",
    "description": "Examples for opening Metapack packages from a CKAN repository",
    "draft": false,
    "github": "https://github.com/sandiegodata/kb.sandiegodata.org/blob/master/notebooks/tutorial/Metapack%20Access%20Example.ipynb",
    "section": "notebooks",
    "show_input": "show",
    "slug": "metapack-access-example",
    "title": " Metapack Access Example",
    "toc": false
}




[Metatab](http://metatab.org) is a system for documenting data set metadata, which the program [metapack](https://github.com/CivicKnowledge/metapack) uses to create data packages. You can also us the metapack python module to access data packages from the web, which provides easy access to documentation and pandas data frames in Jupyter notebooks. 

To start, you'll need to install metatab, which you should be able to do with:

> pip install metatab

The system is under active development, so if there are problems, you can install the latest development versions of the important modules with the [development requirements.txt file on github:](https://github.com/CivicKnowledge/metapack/blob/master/dev/requirements.txt) 

> pip install -r https://raw.githubusercontent.com/CivicKnowledge/metapack/master/dev/requirements.txt

After installing the module, you should be able to run the code in this notebook. 

# Metapack Packages

Metapack packages are collections of files that contain data and metadata. Metapack has several package types, including [Excel](http://s3.amazonaws.com/library.metatab.org/aspe.hhs.gov-dementia_prevalence-2.xlsx), [Zip](http://s3.amazonaws.com/library.metatab.org/aspe.hhs.gov-dementia_prevalence-2.zip), CSV, File systen and S3. Most of the time with Jupyter notebooks, you will use the CSV packages, but the ZIP and Excel packages will also work. 

First, you'll need to get a reference to a package. Most often, you'll get these from our (CKAN Data Repository at data.sandiegodata.org)[http://data.sandiegodata.org]. In this example, well use the [Community Reinvestment Act Disclosure Files](http://data.sandiegodata.org/dataset/ffiec-gov-cra_disclosure_smb_orig-2010_2015). 

First, visit the [data package page in the data repository](http://data.sandiegodata.org/dataset/ffiec-gov-cra_disclosure_smb_orig-2010_2015). The files list will have both data package files and data files. The data package files are the ones that start with the name of the package, `ffiec.gov-cra_disclosure_smb_orig-2010_2015-2`. SO, these are package files: 

* `ffiec.gov-cra_disclosure_smb_orig-2010_2015-2.csv`
* `ffiec.gov-cra_disclosure_smb_orig-2010_2015-2.zip`
* `ffiec.gov-cra_disclosure_smb_orig-2010_2015-2.xlsx`


The last one, the `.csv` file, is the CSV package. Using CSV packages is usually most efficient because you only need to download the data files that you use. So, the first step is to get the CSV package URL. From the [data package page on the CKAN repository](http://data.sandiegodata.org/dataset/ffiec-gov-cra_disclosure_smb_orig-2010_2015) you can:

1. Click on the "Explore" button next to the CSV package file, then right-click on "Go to resource" to copy the URL. 
2. Click on the name of the CSV package, then copy the URL at the top of the following page. 

After you have the package URL, pass it into the `open_package` function, as shown in next cell. The function will return a data package object, which Jupyter will print by showing the package documentation. 


```python

import metapack as mp

pkg =  mp.open_package('http://library.metatab.org/ffiec.gov-cra_disclosure_smb_orig-2010_2015-2.csv')

pkg
```




<h1>Community Reinvestment Act Disclosure Files</h1>
<p><p>ffiec.gov-cra_disclosure_smb_orig-2010_2015-2</p>
<p>Multi-year CRA disclosures for small business originations.</p>
<p>metapack+http://library.metatab.org/ffiec.gov-cra_disclosure_smb_orig-2010_2015-2.csv</p></p>
<h2>Contacts</h2>
<p><strong>Wrangler:</strong> <a href="mailto:eric@sandiegodata.org">Eric Busboom</a> <a href="http://sandiegodata.org">San Diego Regional Data Library</a></p>
<h2>Resources</h2>
<p><ol>
<li><p><strong>sb_loan_orig</strong> - <a target="_blank" href="http://library.metatab.org/ffiec.gov-cra_disclosure_smb_orig-2010_2015-2/data/sb_loan_orig.csv">http://library.metatab.org/ffiec.gov-cra_disclosure_smb_orig-2010_2015-2/data/sb_loan_orig.csv</a> Table D1-1, small business disclosire records, for years 2010 to 2015 inclusive</p></li>
</ol></p>
<h2>References</h2>
<p><ol>
<li><p><strong>discl_15</strong> - <a target="_blank" href="https://www.ffiec.gov/cra/xls/15exp_discl.zip">https://www.ffiec.gov/cra/xls/15exp_discl.zip</a> </p></li>
<li><p><strong>discl_14</strong> - <a target="_blank" href="https://www.ffiec.gov/cra/xls/14exp_discl.zip">https://www.ffiec.gov/cra/xls/14exp_discl.zip</a> </p></li>
<li><p><strong>discl_13</strong> - <a target="_blank" href="https://www.ffiec.gov/cra/xls/13exp_discl.zip">https://www.ffiec.gov/cra/xls/13exp_discl.zip</a> </p></li>
<li><p><strong>discl_12</strong> - <a target="_blank" href="https://www.ffiec.gov/cra/xls/12exp_discl.zip">https://www.ffiec.gov/cra/xls/12exp_discl.zip</a> </p></li>
<li><p><strong>discl_11</strong> - <a target="_blank" href="https://www.ffiec.gov/cra/xls/11exp_discl.zip">https://www.ffiec.gov/cra/xls/11exp_discl.zip</a> </p></li>
<li><p><strong>discl_10</strong> - <a target="_blank" href="https://www.ffiec.gov/cra/xls/10exp_discl.zip">https://www.ffiec.gov/cra/xls/10exp_discl.zip</a> </p></li></p>
</ol>



The `Resources` section lists the datafiles in the package, while the `References` section show the links to datafiles that were used to create the resources. You can use the name of a resource in a call to `pkg.resource` to create a resource object, which like the package object, can be pretty printed in Jupyter. 

```python
r = pkg.resource('sb_loan_orig')
r

```




<h3><a name="resource-sb_loan_orig"></a>sb_loan_orig</h3><p><a target="_blank" href="http://library.metatab.org/ffiec.gov-cra_disclosure_smb_orig-2010_2015-2/data/sb_loan_orig.csv">http://library.metatab.org/ffiec.gov-cra_disclosure_smb_orig-2010_2015-2/data/sb_loan_orig.csv</a></p><table>
<tr><th>Header</th><th>Type</th><th>Description</th></tr><tr><td>table_id</td><td>text</td><td>Value is D1-1</td></tr> 
<tr><td>respondent_id</td><td>text</td><td>Assigned by regulatory agency 
(same as HMDAID if applicable);  
Right justified with leading zeros</td></tr> 
<tr><td>agency</td><td>integer</td><td>Values are 1=OCC, 2=FRS, 
3=FDIC, or 4=OTS</td></tr> 
<tr><td>year</td><td>integer</td><td>Four digit year (e.g. 2012)</td></tr> 
<tr><td>loan_type</td><td>integer</td><td>Value is 4 (Small Business)</td></tr> 
<tr><td>action</td><td>integer</td><td>Value is 1 (Originations)</td></tr> 
<tr><td>state</td><td>integer</td><td>FIPS code with leading zeros 
or blank for totals across all 
states</td></tr> 
<tr><td>county</td><td>integer</td><td>FIPS code with leading zeros or blank 
for totals across all counties</td></tr> 
<tr><td>msa</td><td>integer</td><td>As defined by OMB; Right justified 
with leading zeros, NA left justified 
for areas outside of MSA/MD or blank 
for totals across all MSA/MDs</td></tr> 
<tr><td>assessment_area</td><td>text</td><td>Values are 0001 through 9999; Right 
justified with leading zeros, NA left justified 
for areas outside of an Assessment 
Area (including predominately military 
areas) OR blank for totals across 
all Assessment Areas</td></tr> 
<tr><td>partial_county</td><td>text</td><td>Values are 
Y = Yes 
N = No 
OR blank for totals</td></tr> 
<tr><td>split_county</td><td>text</td><td>Values are 
Y = Yes 
N = No 
OR blank for totals</td></tr> 
<tr><td>pop_class</td><td>text</td><td>Values are 
S= counties with 
< 500,000 in population 
L= counties with 
>500,000 in population 
OR blank for totals</td></tr> 
<tr><td>income_total</td><td>text</td><td>Values are 
1= < 10% of Median Family 
Income(MFI) 
2= 10% to 20% of MFI 
3= 20% to 30% of MFI 
4= 30% to 40% of MFI 
5= 40% to 50% of MFI 
6= 50% to 60% of MFI 
7= 60% to 70% of MFI 
8= 70% to 80% of MFI 
9= 80% to 90% of MFI 
 10= 90% to 100% of MFI 
 11= 100% to 110% of MFI 
 12= 110% to 120% of MFI 
 13= > 120% of MFI 
 14= MFI not known (income 
percentage = 0) 
 15= Tract not Known (reported 
as NA) 
101= Low Income (< 50% of 
MFI - excluding 0) 
102= Moderate Income (50% 
to 80% of MFI) 
103= Middle Income (80% to 
120% of MFI) 
104= Upper Income (> 120% 
of MFI) 
105= Income Not Known (0) 
106= Tract not Known (NA) 
Right justified with leading zeros 
or blank for totals</td></tr> 
<tr><td>report_level</td><td>text</td><td>Values are 
4= Total Inside & Outside 
Assessment Area (AA) 
(across all states) 
6= Total Inside AA 
(across all states) 
8= Total Outside AA 
(across all states) 
  10= State Total 
  20= Total Inside AA in State 
  30= Total Outside AA in State 
  40= County Total 
  50= Total Inside AA in County 
  60= Total Outside AA in County 
Right justified with leading zeros 
or blank if not a total</td></tr> 
<tr><td>num_orig_bus_lt100k</td><td>integer</td><td>Right justified with leading zeros</td></tr> 
<tr><td>tot_orig_bus_lt100k</td><td>integer</td><td>Amount is in thousands {e.g. 00000025 
indicates $25,000); Right justified with 
leading zeros</td></tr> 
<tr><td>num_orig_bus_lt250k</td><td>integer</td><td>Right justified with leading zeros</td></tr> 
<tr><td>tot_orig_bus_gt100k_lt250k</td><td>integer</td><td>Amount is in thousands {e.g. 00000125 
indicates $125,000); Right justified with 
leading zeros</td></tr> 
<tr><td>num_orig_bus_gt250k_lt1m</td><td>integer</td><td>Right justified with leading zeros</td></tr> 
<tr><td>tot_orig_bus_gt250k_lt1m</td><td>integer</td><td>Amount is in thousands {e.g. 00000300 
indicates $300,000); Right justified with 
leading zeros</td></tr> 
<tr><td>num_orig_bus_lt1m</td><td>integer</td><td>Right justified with leading zeros</td></tr> 
<tr><td>tot_orig_bus_lt1m</td><td>integer</td><td>Amount is in thousands {e.g. 00000025 
indicates $25,000); Right justified with 
leading zeros</td></tr> 
<tr><td>num_orig_bus_al</td><td>integer</td><td>Right justified with leading zeros</td></tr> 
<tr><td>tot_orig_bus_al</td><td>integer</td><td>Amount is in thousands {e.g. 00000025 
indicates $25,000); Right justified with 
leading zeros</td></tr> </table>



The final step of access is to create a dataframe from the resource. This is really easy, just use the `.dataframe()` method. Note, however, for this dataset, it can take almost 10 minutes to create the whole dataframe, as the data file is very large. 

```python
%%time
df = r.dataframe()
```

    CPU times: user 9min 7s, sys: 8.66 s, total: 9min 16s
    Wall time: 9min 36s


```python
len(df)
```




    4391391


