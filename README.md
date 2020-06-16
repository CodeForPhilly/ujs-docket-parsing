# ujs-docket-parsing

**This project needs project / product managers, UX, and engineering support!**

## What is this project about?

Currently, there are many legal organizations that need to pull information from openly available court dockets.
However, some formatting quirks make parsing this data is tough.
This project is focused on overcoming these quirks and parsing the dockets.

Here's how some organizations would benefit from parsing this data:

* [Community Legal Services](https://codeforphilly.org/projects/tools_for_sealing_and_expunging_criminal_records_in_pennsylvania) - could quickly check which records can be [sealed](https://clsphila.org/highlights/clean-slate-2/).
* [Philadelphia Lawyers for Social Equity](https://codeforphilly.org/projects/philadelphia_lawyers_for_social_equity_-_record_expungement) - could quickly identify which records their clients could have [expunged](https://www.plsephilly.org/expungements/).
* Philadelphia Bail Fund - could analyze and report bail figures over the past year, broken down into more detail.


### Key challenges

* Docket information is **sensitive**, so should not be stored in this repo.
* Edge cases are **unknown** until we find them, so we need to collect test cases for parsers.
* Testing parsers requires a strategy for **feeding** them records and **evaluating** their outputs.

### References

* [Issues in @NateV's recordlib](https://github.com/CLSPhila/RecordLib/issues) - mention problematic dockets.
* [Folder in PLSE's repo](https://github.com/Philadelphia-Lawyers-for-Social-Equity/etl-db-env/blob/master/etl/) - other attempt at parsing dockets.



## Downloading and parsing dockets

In general, it's easiest to download dockets using a headless browser.
They are often the parsed by first converting the docket pdf to text, before feeding to a formal parser or regex.

### Downloading a docket

First, clone and start @NateV's DocketScraperAPI:

```
git clone https://github.com/CLSPhila/DocketScraperAPI.git utils/DocketScraperAPI
cd utils/DocketScraperAPI && docker-compose -f dev-compose up
```

This allows you to search for and download dockets.
Below is an example of using python's requests library to search.


```python
import requests

COURT_NAME = "MDJ"
API_URL = "http://localhost:8800"
WEB_HEADERS = {'User-Agent': 'Mozilla/5.0 (Macintosh; Intel Mac OS X 10_10_1) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/39.0.2171.95 Safari/537.36'}

# Make request to searchName ----
person = {
        "first_name": "Kathleen",
        "last_name": "Kane",
        "dob": "06/14/1966"
        }

# will return multiple dockets, including partial name matches
r = requests.post(
        API_URL + "/searchName/" + COURT_NAME,
        json = person
        )

r.json()
```

Here is an example of retrieving a docket. For context on this docket. 

```python
import requests

COURT_NAME = "CP"
API_URL = "http://localhost:8800"
WEB_HEADERS = {'User-Agent': 'Mozilla/5.0 (Macintosh; Intel Mac OS X 10_10_1) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/39.0.2171.95 Safari/537.36'}

FILE_NAME = "docket_example.pdf"

# Make request to lookupDocket ----
r2 = requests.post(
        API_URL + "/lookupDocket/" + COURT_NAME,
        json = {"docket_number":  "CP-46-CR-0006239-2015"}
        )

data = r2.json()
docket_url = data['docket']['docket_sheet_url']


# Fetch docket, and save as pdf
r_docket  = requests.get(docket_url, headers = WEB_HEADERS)

with open(FILE_NAME, 'wb') as f:
    f.write(r_docket.content)
```

For context on this docket, see this [New York Times article](https://www.nytimes.com/2016/08/16/us/trial-kathleen-kane-pennsylvania-attorney-general.html).


### Parsing a docket

An example was provided by Pablo Virgo, and has been included in `examples/simple_parser`.

You can either install the dependencies and run directly..

```python
pip install -r examples/simple_parser/requirements.txt
python examples/simple_parser/parser.py docket_example.pdf
```

Or can build and run the parser using Docker.

```bash
docker build ./examples/simple_parser -t parser

docker run --rm -v "$PWD":"/files" parser python parser.py /files/docket_example.pdf
```

