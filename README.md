# Inception
**Inception** is a highly configurable tool to check for whatever you like against any number of hosts. It is extremely easy to use, yet highly configurable. It only takes few minutes to create fingerprints/signatures of the desired vulnerability/service one wants to scan by using [providerCreate.html](https://proabiral.github.io/inception/providerCreate.html) .

This tool comes handy for bugbounty hunters who want to check for specific endpoint on large number of hosts and report if the endpoint returns the specified String, Status Code or Content-Length.

Inception is a Go version of [Snallygaster](https://github.com/hannob/snallygaster) and comes with a large number of test cases derived from Snallygaster plus more, added by me.    

Default test cases includes: test for publicly accesible git config file, .env file, magento config file, php info file, server stats page, Rails and Symfony database config files, CORS Misconfiguration check, basic XSS check at web root and few others.    

What differentiate Inception from Snallygaster is - Inception allows users to create & provide their own test cases without touching a single line of code.

The use of goroutine makes it very fast but it doesn't hammer a single domain concurrently with a large number of requests as single test case is performed against all the specified domain before moving to another test case.

### Key Features
1) Very easy to use and create signatures using [providerCreate.html](https://proabiral.github.io/inception/providerCreate.html)
2) Extremely fast due to use of goroutine
3) Does not hammer single server and prevent WAF blocking
4) Highly configurable to send GET/POST/HEAD/PUT/DELETE request with desired headers and body.
5) Minimum False Positive as Response can be filered with specific String, Status code, Content Length.

### Installation
Just make sure you have go installed and run the following command.
```sh
go get github.com/proabiral/inception
```

### Update
```sh
go get -u github.com/proabiral/inception
```

### Usage
```
▶️  inception -h
    Usage of inception:
  -caseSensitive
        case sensitive checks
  -d string
        Path of list of domains to run against (default "/home/proabiral/go/src/github.com/proabiral/inception/domains.txt")
  -https
        force https (works only if scheme is not provided in domain list
  -noProgressBar
        hide progress bar
  -o string
        File to write JSON result
  -provider string
        Path of provider file (default "/home/proabiral/go/src/github.com/proabiral/inception/provider.json")
  -silent
        Only prints when issue detected
  -t int
        No of threads (default 200)
  -timeout int
        HTTP request Timeout (default 10)
  -v    Verbose mode

```

#### Basic Example
```
▶️ inception -d /path/to/domainlist.txt
Issue detected : Server status is publicly viewable http://127.0.0.1/server-status response contains all check
Issue detected : PHP info is publicly viewable http://127.0.0.1/phpinfo.php response contains all check
Completed
```
All detected issues will be printed on screen as shown above. While if no issue is detected, a completion message is shown as `Completed`.    
Note: If error like `provider.json: no such file or directory` is thrown, provide the path of provider.json {default one located at your-gopath/src/github.com/proabiral/inception/provider.json} file with -provider option.    

#### Saving Output to File
```
▶️ inception -d /path/to/domainlist.txt -o /tmp/output
```

Output is saved in json format using can easily be filtered out later using tools like <a href="https://stedolan.github.io/jq/">jq</a> and <a href="https://github.com/tomnomnom/gron">gron</a>.
   
### FAQs
Q. How should my domain list look like?    
A sample of domain list is provided with the tool. It's basically a list of line seperated domains without no protocol.
```
facebook.com
twitter.com
gmail.com
hackerone.com
bugcrowd.com
```

Q. How do I add my own test cases?    
You can use [providerCreate.html](https://proabiral.github.io/inception/providerCreate.html) to generate JSON. Just fill in the details and JSON as shown below will be generated.
```
[
       {
           "vulnerability": "Server status is publicly viewable",
           "method": "GET",
           "color": "blue",
           "body": "",
           "endpoint": [
               "/server-status"
           ],
           "headers": [],
           "checkIn": "responseBody",
           "checkFor": "CPU Usage&&&&Server Version&&&&Apache Server Status"
       },    
       {
           "vulnerability": "Tomcat Server status is publicly viewable",
           "method": "GET",
           "color": "blue",
           "body": "",
           "endpoint": [
               "/status?full=true"
           ],
           "headers": [],
           "checkIn": "responseBody",
           "checkFor": "Current thread count"
       }
]

```

Save the generated JSON to some file and then run the tool by providing the path to the json file with `-provider` option:
```
▶️  inception -provider /path/to/your/provider.json -d /path/to/your/domainlist.txt
```

#### Placeholders in Providers
You can also use the following predefined placeholders in provider.json which will get replaced according to the URL against which the tool is ran.
* $hostname
* $domain 
* $fqdn

For example, if you're running inception against www.example.com

$hostname becomes example  <br />
$domain becomes example.com and <br />
$fqdn becomes www.example.com <br />

This is very helpful for cases like CORS. Or if you want to check for files prefixed by domain name or alike.

Below is the example showing fingerprint and request generated by it against `https://end8ej99anfko.x.pipedream.net`

```
[ 
    {
        "vulnerability": "Demo Bug",
        "method": "GET",
        "color": "blue",
        "body": "fqdn=$fqdn&domain=$domain&hostname=$hostname",
        "endpoint": [
            "/test?FQDN=$fqdn&domain=$domain&hostname=$hostname"
        ],
        "headers": [
            [
                "FQDN",
                "$fqdn"
            ],
            [
                "domain",
                "$domain"
            ],
            [
                "hostname",
                "$hostname"
            ]
        ],
        "checkIn": "responseBody",
        "regexCheck": false,
        "checkFor": ""
    }
]
```

<img src="https://raw.githubusercontent.com/proabiral/inception/master/images/placeholder.png" alt="incoming request" height="80%" width="80%">



Q. Whats with the name?    
The name of tool is inspired from the movie Inception where DiCaprio steals secrets from subconscious mind of people. Similar to movie, this tool steal secrets from webserver.    
Also, `inception` because this is the first tool I am open sourcing.

## Thanks 
Thanks to [Iceman](https://twitter.com/Ice3man543) for reviewing the tool and suggesting this cool name.
Also concurrency module has been shamelessly stolen from his [Subover project](https://github.com/Ice3man543/SubOver)

