# Java HTTPS Tester

A small program to help you debug HTTPS connection problems with
JVM-based programs.

## Motivation

In
[Java 1.8.0_141](http://www.oracle.com/technetwork/java/javase/8u141-relnotes-3720385.html) Oracle
removed support for SHA-1 signed certificates in HTTPS
connections. We've seen connection problems after such a Java update
in production systems when not all certificates where updated before
the Java update was rolled out. Another reason for connection problems
we often see is the missing installation of the strong crypto for Java
(JCE, see below).

This program can be used to test and analyze an HTTPS connection
established by a JVM-based program. You can test your local Java
installation and your server-settings with this.

## Build

You need to install [Leiningen](https://leiningen.org/). Don't worry,
it's really easy. Just a bash script in your path on Linux, OSX, or
Cygwin. There is also a `.bat`` file for Windows.

To create the executable JAR, just run

```bash
shell> lein uberjar
```

in the cloned repository. The resulting executable JAR will be in the
`target` folder.

## Usage

You can simply call `lein run` with a URL to connect to:

```bash
lein run https://www.example.com
```

or run the executable JAR:

```bash
java -jar target/java-https-tester.jar https://www.example.com
```

This program will connect to the server at least two times:

* Once using the Apache HTTP Commons library,
* and once using the built-in classes of the JRE.

It prints the name of a logfile where all information is
collected. The logfile's contents may be a lot to digest, but look at
it line by line and you'll see that it is pretty readable.

This program does not use the Apache library directly, but via the
Clojure library [clj-http](https://github.com/dakrone/clj-http). The
`get` function of that library allows some configurations.  We already
pass in

```clojure
    {:throw-exceptions true
     :debug true
     :response-interceptor interceptor}
```

which cannot be overwritten.  The `interceptor` should output some
logs when you follow redirects.  For other things like proxy
configurations, keystores, or basic auth, see the documentation
at [clj-http](https://github.com/dakrone/clj-http).

Pass the configuration map as
an [EDN](https://github.com/edn-format/edn) formatted string
parameter:

```bash
java -jar java-https-tester.jar -c '{:proxy-host "127.0.0.1"  :proxy-port 8118}' URL
```

When connecting to the URL the second time using the common Java
classes, you can control its behavior by setting the appropriate
system properties.

For example, you could create extensive SSL debug logging by calling
the program like this:

```bash
java -Djavax.net.debug=ssl -jar java-https-tester.jar URL
```

See [Debugging SSL/TLS Connections](
https://docs.oracle.com/javase/7/docs/technotes/guides/security/jsse/ReadDebug.html)
and
[Customizing JSSE](https://docs.oracle.com/javase/8/docs/technotes/guides/security/jsse/JSSERefGuide.html#InstallationAndCustomization)
for more.

If the connection could not be established so far, often because of
SSL handshake errors, the program tries a third time using an all
trusting trust manager.  If this succeeds, you'll find the certificate
chain information in the log file, too.

If still no connection can be established, this is often a good hint
that you should install JCE.

## Hints

Some hints what you want to look for in such a debugging situation.
Your mileage may vary.

* The most thorough document for everything related to this topic is
  probably the
  [JSSE Reference Guide](http://docs.oracle.com/javase/8/docs/technotes/guides/security/jsse/JSSERefGuide.html).
* Find the file `java.security` in your `JAVA_HOME/jre/lib/security`
  folder. There is a setting called `jdk.certpath.disabledAlgorithms`
  which was changed significantly in Java 1.8.0_141. It restricts
  usage of SHA-1-signed certificates for TLS connections. Compare it
  to older versions and adjust if you absolutely have to. But be
  warned that you are reducing security by doing so. Always prefer
  using suitable certificates on the server-side.
* In some cases the `cacerts` file in that same folder caused problems
  for us.
* Depending on the configuration of the server you are connecting to,
  you may have to install
  the
  [Java Cryptography Extension (JCE) Unlimited Strength Jurisdiction Policy](http://www.oracle.com/technetwork/java/javase/downloads/jce8-download-2133166.html).
  Otherwise some ciphers will not be available.
* If you are installing Oracle Java 8 on a developer Linux machine
  using the [webupd8 PPA](https://launchpad.net/~webupd8team), you may
  find out that you did not receive Oracle's changes with an
  update. The package manages some of the files in the `security`
  folder via symlinks to `/etc`.  Analyze those carefully.
* Another, much more elaborate, tool is the `s_client` module from the
  OpenSSL project. It can validate and output certificates and help a
  lot with debugging connection problems. It does not use the JVM
  though, and is thus a different beast. You often use it like this:

```bash
openssl s_client -showcerts -connect www.example.com:443
```

## Support

This is not an Acrolinx product. It is a helper tool and provided as
is. We do not support this program.

## License

Copyright 2017-present Acrolinx GmbH

Licensed under the Apache License, Version 2.0 (the "License");
you may not use this file except in compliance with the License.
You may obtain a copy of the License at:

[http://www.apache.org/licenses/LICENSE-2.0](http://www.apache.org/licenses/LICENSE-2.0)

Unless required by applicable law or agreed to in writing, software
distributed under the License is distributed on an "AS IS" BASIS,
WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
See the License for the specific language governing permissions and
limitations under the License.

For more information visit: [https://www.acrolinx.com](https://www.acrolinx.com)
