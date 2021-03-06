hippo7-rproxy-nodejs
====================

Node.js script for Reverse Proxy targeting Hippo CMS 7 and HST-2 applications.
You can run this simple script as a reverse proxy server supporting both HTTP and HTTPS.

Note
----
The current version of rproxy.js wohttrks with the latest versions (1.0.x) of 'http-proxy' module.
The old version of rproxy.js which worked with the outdated versions (0.8.x or earlier) of 'http-proxy' module has moved to 'with-http-proxy-0.8.x' branch.

Introduction
------------

Hippo CMS 7 (http://www.onehippo.org) is a Java based Open Source Web Content Management Solution.
Hippo CMS 7 solution usually consists of multiple web applications and 
system administrators often deploy a reverse proxy server before Java application servers for many reasons.

Apache HTTP Server with mod_proxy is a popular solution for the reverse proxy node, but it is not so convenient
to install Apache HTTP Server on a developer's computer.

So, I looked for an alternative solution for convenience of developers who want to test in the same environment as
the production server.

The solution is <a href="http://nodejs.org/">Node.js</a>!
Yes, I was able to implement a full-featured, reliable reverse proxy script with Node.js very quickly.

How to run the reverse proxy server script
------------------------------------------

  Note: First, you need to install <a href="http://nodejs.org/">Node.js</a> in order to run Reverse Proxy Server script.
        See the next section for installation guide.

  1. Copy the following two files into the root folder of your Hippo CMS 7 project.
    * <a href="https://raw.github.com/woonsan/hippo7-rproxy-nodejs/hippo7-rproxy-nodejs-1.0.x/rproxy.js">rproxy.js</a>
    * <a href="https://raw.github.com/woonsan/hippo7-rproxy-nodejs/hippo7-rproxy-nodejs-1.0.x/package.json">pakage.json</a>

      Or you can run the following in the command line to download it:

      <pre>
        $ curl -L https://raw.github.com/woonsan/hippo7-rproxy-nodejs/hippo7-rproxy-nodejs-1.0.x/rproxy.js > rproxy.js
        $ curl -L https://raw.github.com/woonsan/hippo7-rproxy-nodejs/hippo7-rproxy-nodejs-1.0.x/package.json > package.json
      </pre>

  2. Install dependencies (you can run this only once).

    Run the following command in the root folder of your Hippo CMS 7 project:
    
    <pre>
      $ npm install
    </pre>
    
    (The above command will install all the dependencies defined in the package.json.)

    * Note: if you have a problem in installing with npm due to a corporate proxy, then please follow
            Wil Boayue's blog article: [Using Npm Behind a Corporate Proxy] (http://wil.boayue.com/blog/2013/06/14/using-npm-behind-a-proxy/).

  3. Move to the root folder of your Hippo CMS 7 project in the command line console and run the following command:

     <pre>
       $ sudo node rproxy.js
     </pre>

     * Note: Windows Users can omit `sudo' for the above and the other command examples below:
     <pre>
       C:\\...> node rproxy.js
     </pre>

    The above command will run the Reverse Proxy Server at port 80 and at SSL port 443 by default.
    (The SSL port will open only if you have SSL private key file and certificate file properly.
    The default paths are './priv.pem' and './cert.pem'.)

    You can run it at different ports like the following example:

     <pre>
        $ sudo node rproxy.js 8888
     </pre>
    
    The above command will open 8888 for HTTP and 443 for HTTPS.
    
     <pre>
        $ node rproxy.js 8888 8443
     </pre>
    
    The above command will open 8888 for HTTP and 8443 for HTTPS.

  4. For your information, to generate SSL private key and certificate files for testing purpose,
     you can simply run the following:

<pre>
    $ openssl genrsa -out priv.pem 1024 
    $ openssl req -new -key priv.pem -out cert.csr 
    $ openssl x509 -req -in cert.csr -signkey priv.pem -out cert.pem
</pre>

About installing Node.js
----------------------------------------------

  Download a Node.js installer package from the following site and install it on your system.
  - http://nodejs.org/

Reverse Proxy Mapping Configuration
-----------------------------------

  If you open the 'rproxy.js' file, you can find the following as the default mapping configuration:
  
<pre>
  /**************************************************************/
  /* URL Path Mappings for Reverse Proxying to Target Servers */
  /**************************************************************/
  /* You can add edit mappings below! */

  var mappings = [
    {
      host: '*',
      pathregex: /^\/cms(\/|$)/,
      route: {
        target: 'http://localhost:8080'
      }
    },
    {
      host: '*',
      pathregex: /^\/site(\/|$)/,
      route: {
        target: 'http://localhost:8080'
      }
    },
    {
      host: '*',
      pathregex: /^/,
      pathreplace: '/site',
      setcookie: {
        pathregex: /^\/site/,
        pathreplace: '/'
      },
      route: {
        target: 'http://localhost:8080'
      }
    },
  ];
</pre>

  Based on the 'mappings' array object, the 'rproxy.js' will do reverse proxying for each target.
  In the above default example mappings, there are two mappings defined.
  
  The first mapping checks if the request host header matches the host property ('*' means any host),
  and if the request path is just '/cms' or starts with '/cms/' (e.g., http://localhost:8080/cms/...).
  If the request path is just '/cms' or starts with '/cms/', then it routes the request to the target, 'http://localhost:8080'.
  And, in this case, the request path doesn't change.
  So, the request path like '/cms/...', will be targeted to 'http://localhost:8080/cms/...'.
  
  The third mapping checks if the request host header matches the host property ('*' means any host),
  and if the request path starts with *any* (by the regular expression, /^/), then
  it prepends the request path by '/site' and the request is targeted to the 'http://localhost:8080'.
  So, the request path like '/news/...', will be targeted to 'http://localhost:8080/site/news/...'.
  'pathreplace' property is optional. If it's defined, then it is used to replace the found match by 'pathregex' property.

  The second mapping was added only to allow requests having the same context path, which is targeting to the same one
  as the third one explained above. (You may remove this if you don't need to allow this.)

  You can add or remove mapping in 'mappings' object just by editing 'rproxy.js'.

Changing the default SSL private key and certificate file paths
---------------------------------------------------------------

  If you want to use different SSL private key and certificate file paths, then open the 'rproxy.js' file.
  You can find the following as the default file paths:
  
<pre>
/*============================================================*/
/* Reverse Proxy Server Default Options Configuration         */
/*------------------------------------------------------------*/

var defaultOptions = {
  xfwd: true, // add X-Forwarded-* headers
};

// SSL Key file paths; change those paths if you have those in other paths.
var ssl_private_key_path = './priv.pem';
var ssl_certificate_path = './cert.pem';
</pre>

  Change the file paths above to whatever you want to use.

Simulating Multiple Domain Environment on your computer
-------------------------------------------------------

  Sometimes we want to test multiple domain environment for some reason. e.g., client-side federation between domains.
  You can use multiple domains with this rproxy.js to simulate multiple domains environments.
  For example, you can configure the mapping to let 'http://www1.example.lan' go to 'http://localhost:8080/site',
  while 'http://www2.example.lan' go to 'http://localhost:9080/site'.
  The following is an example mapping configuration for this scenario:
  
<pre>
/*------------------------------------------------------------*/
/* URL Path Mappings Configuration for Reverse Proxy Targets  */
/*------------------------------------------------------------*/
// You can add edit mappings below!

  var mappings = [
    {
      host: 'www1.example.lan',
      pathregex: /^/,
      pathreplace: '/site',
      route: {
        target: 'http://localhost:8080'
      }
    },
    {
      host: 'www2.example.lan',
      pathregex: /^/,
      pathreplace: '/site',
      route: {
        target: 'http://localhost:9080'
      }
    },
  ];
</pre>
  
  Of course, you will need to add those host names into your hosts file (e.g., /etc/hosts) like this:

<pre>
127.0.0.1    www1.example.lan
127.0.0.1    www2.example.lan
</pre>

  Then you don't have to prepare two different machines, but you can focus more on your federated scenario implementation/testing!

Adding Custom Request Headers on Specific Proxy Mapping
-------------------------------------------------------
You can also add custom proxy request headers by setting 'reqHeaders' property like the following examples:
<pre>
/*------------------------------------------------------------*/
/* URL Path Mappings Configuration for Reverse Proxy Targets  */
/*------------------------------------------------------------*/
// You can add edit mappings below!

  var mappings = [
    {
      host: 'www1.example.lan',
      pathregex: /^/,
      pathreplace: '/site',
      route: {
        target: 'http://localhost:8080'
      },
      reqHeaders: {
        'X-Special-Proxy-Header': 'foobar'
      }
    },
  ];
</pre>


-----

OK. Now enjoy working with rproxy.js (powered by Node.js) !!

Cheers!
