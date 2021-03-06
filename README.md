# accesshandler

Resource access limit handler

### What does it do?

The **Access Handler** is a web service that stores the _Rules_ user creates.
The _Rules_ contains:

- pattern: It can be either an exact explicit url or a regex
- limit: Shows how many times an specific IP can request for an url matched
  the pattern

Sample rule be like:

```json
{
    "limit":"100/sec",
    "isExactUrl":false,
    "pattern":"/foo/.*",
    "id":1
}
```

Also this web service recieves logs by the format of:

```json
{
  "url": "example.com/foo",
  "IP": "1.1.1.1"
}
```

showing the which `IP` requested for `url`.

The goal is to check if an specific `IP` requested more than limit time interval
for specific `url` matched the patterns and inform by `429 Too Many Requests`
HTTP status.

### In action

Let's test how it works step by step:

**NOTE:** the `$URL` is url of web service.

1. Create a rule by [CREATE rule API](API.md#creating-an-rule) and set the 
limit to a value like `2/min`, like:

```bash
curl -X CREATE -H "Content-Type: application/json" --data '{"pattern": "example.com/foo/.*", "limit": "2/min"}' -i -- "$URL/apiv1/rules"
```

2. Try to post log by [POST log API](API.md#post-a-log-to-check-if-passed-the-limit-or-not) 
for more than `2` times in less `1` minute. The web service must respond you 
with `200 OK` status for `2` first times, and `429 Too Many Requests` for more
than `2` times of request. Your request should be like below:

```bash
curl -X POST -H "Content-Type: application/json" --data '{"url": "example.com/foo/bar", "IP": "1.1.1.1"}' -i -- "$URL/apiv1/logs?"
```

What we did was creating a rule that shows an specific `IP: 1.1.1.1` can view the 
`url: example.com/foo/bar` at most for `2` times per minute because this url 
matched with created rule `pattern: /foo/.*`.


### Installing dependencies

```bash
sudo apt-get install libass-dev libpq-dev postgresql \
    build-essential redis-server redis-tools
```

### Installing project by pip

**NOTE:** Highly recommended to use `virtual environment`. There are some pip
packages for this purpose. But I offer you using `virtualenvwrapper` package.

You can install by 'pip install' and use https by the following way:

```bash
pip install git+https://github.com/shayan-7/accesshandler.git
```

Or you can use SSH:

```bash
pip install git+git@github.com:shayan-7/accesshandler.git
```

### Installing project (edit mode)

So, your changes will affect instantly on the installed version

#### accesshandler

```bash
cd /path/to/workspace
git clone git@github.com:shayan-7/accesshandler.git
cd accesshandler
pip install -e .
```

### Setup database

#### Configuration

Accesshandler is zero configuration application and there is no extra
configuration file needed, but if you want to have your own
configuration file, you can make a `accesshandler.yml` in the following
path: `~/.config/accesshandler.yml` such as following format:

```yml
db:
  url: postgresql://postgres:postgres@localhost/accesshandler_dev
  test_url: postgresql://postgres:postgres@localhost/accesshandler_test
  administrative_url: postgresql://postgres:postgres@localhost/postgres
```

#### Remove old abd create a new database **TAKE CARE ABOUT USING THAT**

```bash
accesshandler db create --drop --mockup
```

#### Drop old database: **TAKE CARE ABOUT USING THAT**

```bash
accesshandler [-c path/to/config.yml] db drop
```

#### Create database

```bash
accesshandler [-c path/to/config.yml] db create
```

Or, you can add `--drop` to drop the previously created database: **TAKE CARE ABOUT USING THAT**

```bash
accesshandler [-c path/to/config.yml] db create --drop
```

#### Create schema

```bash
accesshandler [-c path/to/config.yml] db schema
```

### Running tests

To check all tests passing and **100%** coverage run the following command:

```bash
pip install -r requirements-ci.txt
pytest --cov=accesshandler
```

Then all the tested APIs generated in `<path/to/accesshandler>/data/markdown`
by Markdown format.

### Serving

- Gunicorn

```bash
$ ./gunicorn
```

### APIs list

Check out [API.md](API.md) for list of APIs and sample responses.
