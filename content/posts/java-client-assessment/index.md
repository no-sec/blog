---
title: Assessing Java Desktop Clients (JAR and JNLP)
date: 2026-08-13
description: A detailed workflow for assessing Java desktop clients with interception, decompilation, debugging, and Java Agent instrumentation.
draft: false
tags: [java, pentest, desktop, reverse-engineering]
toc: true
authors:
  - Falk Huber
---

## Abstract

Java desktop clients are still widespread in enterprise environments and are often launched as standalone JAR files or via Java WebStart / JNLP.  
Unlike browser-centric targets, certificate validation is usually bound to Java truststores rather than the OS trust store, so interception and analysis require a different workflow.

This article provides a practical approach for security assessments:

1. traffic interception for HTTP/S and plain TCP-based channels,
2. decompilation and static mapping of runtime behavior,
3. runtime debugging through JDWP and IntelliJ,
4. Java Agent instrumentation for deeper runtime control.

## Why This Topic Is Tricky

Assessments fail early when testers assume Java desktop clients behave like web browsers.  
In practice, you face truststore divergence, mixed protocols, dynamic endpoint loading, reflection-heavy code paths, and anti-analysis controls.

Typical consequences:

- HTTPS interception fails although the proxy is configured.
- Traffic appears as unreadable binary streams.
- Critical logic is hidden behind runtime-loaded classes.
- Static analysis alone does not reveal effective runtime behavior.

On the other hand, a significant portion of the application's logic resides on the tester's system, thereby placing it under their control. This opens up the possibility of analysing and manipulating this logic, including client-side security controls.

## JNLP and WebStart: Operational Context

A JNLP file is an XML launch descriptor for WebStart applications that are loaded dynamically from a web server. It usually specifies downloaded JARs, start-up parameters, JVM arguments and optional security directives. 
The backend endpoint is often not hardcoded in the launch descriptor and may be fetched dynamically at runtime.
For assessments, this means that the JNLP file, the downloaded configuration artefacts and the decompiled client code are all within scope.

## Scope and Assumptions

- **Platforms:** Windows and Linux
- **Targets:** standalone JAR (`java -jar`) and JNLP/WebStart (`javaws`)
- **Tools:** Burp Suite, mitmproxy, Wireshark, jd-gui, IntelliJ IDEA
- **Out of scope:** Android/mobile Java clients

## Tooling Overview

| Tool | Purpose | Why it matters |
| --- | --- | --- |
| Burp Suite | HTTP/S interception | Best for request/response-level web protocol testing |
| mitmproxy / mitmdump | TCP/TLS/SOCKS interception | Useful when traffic is not pure HTTP |
| Wireshark | Baseline protocol and host discovery | Prevents blind proxy setup |
| jd-gui | Fast decompilation | First pass to identify communication paths |
| IntelliJ IDEA | Remote debugging + source navigation | Runtime truth for control flow and object state |

Linux prerequisite for JNLP:

```bash
sudo apt install icedtea-netx
```

Alternative: OpenWebStart.

---

## Quick Start Checklist

| # | Action | Area |
| --- | --- | --- |
| 1 | Record baseline traffic in Wireshark | Discovery |
| 2 | Decompile JARs and locate endpoint/trust logic | Static analysis |
| 3 | Import proxy CA into effective truststore | TLS interception |
| 4 | Configure JVM proxy properties | Routing |
| 5 | Add mitmproxy for non-HTTP channels | Protocol coverage |
| 6 | Enable JDWP and attach IntelliJ | Runtime inspection |
| 7 | Add Java Agent hooks when required | Runtime manipulation |

---

## 1) Traffic Interception

### Threat Model

Java desktop clients often use one or multiple of the following protocols: HTTP/S, SOAP, REST, RMI, and raw TCP sockets.  
Certificate validation is usually performed by JSSE against Java truststores.  
If the proxy CA is absent from the active truststore, TLS interception fails with errors like:

```text
PKIX path building failed
```

### Truststore Resolution Order

Java commonly resolves trust in this order:

1. Global JRE store (`$JAVA_HOME/lib/security/cacerts`)
2. Application-local truststore (`*.jks`, `*.p12`, or custom-named files)
3. JVM override (`-Djavax.net.ssl.trustStore=<path>`)

### CA Import Strategy

Prefer a dedicated test truststore over global modification to reduce side effects across unrelated Java apps.

Java truststores are essentially keystores, which require a passphase to be set. For truststores (`cacerts`) this is commonly set to `changeit`.

```bash
# Burp CA as DER
keytool -import -trustcacerts -file burpca.der -alias BURPSUITE -keystore cacerts -storepass changeit
```

Windows global variant:

```bash
keytool -import -trustcacerts -file burpca.der -alias BURPSUITE -keystore "%JAVA_HOME%\lib\security\cacerts" -storepass changeit
```

### JVM Proxy Arguments for Standalone JAR

In order to use the proxy, such as Burp Suite, the Java client needs to set the following JVM arguments:

```bash
java -Dhttp.proxyHost=localhost -Dhttp.proxyPort=8080 \
     -Dhttps.proxyHost=localhost -Dhttps.proxyPort=8080 \
     -Djavax.net.ssl.trustStore=cacerts \
     -Djavax.net.ssl.trustStorePassword=changeit \
     -jar app.jar
```

### JVM Proxy Arguments for JNLP/WebStart

```bash
# Windows
set JAVAWS_VM_ARGS=-Dhttp.proxyHost=localhost -Dhttp.proxyPort=8080 -Dhttps.proxyHost=localhost -Dhttps.proxyPort=8080

# Linux
export JAVAWS_VM_ARGS="-Dhttp.proxyHost=localhost -Dhttp.proxyPort=8080 -Dhttps.proxyHost=localhost -Dhttps.proxyPort=8080"
```

IcedTea-Web GUI fallback:

```bash
javaws -viewer
```

### Fallback When `-Dhttp*` / `-Dhttps*` Are Ignored

Some clients clear proxy properties, use custom selectors, or rely on runtime OS proxy lookup.  
In such cases, this fallback can help:

```bash
java -Djava.net.useSystemProxies=true -jar app.jar
```

It routes through system proxy settings but does **not** solve trust by itself.

### Where Proxy/Trust Arguments May Be Injected

| Location | Use case | Notes |
| --- | --- | --- |
| Java command line | Standalone JAR | Most direct and transparent |
| `JAVAWS_VM_ARGS` | JNLP launchers | Passed by `javaws` to target JVM |
| Wrapper scripts | Vendor-distributed launchers | Check `.bat`/`.sh` |
| App config files | Some enterprise clients | No `-D` prefix in config keys |
| GUI settings | User-configurable clients | Often overlooked during tests |

### Non-HTTP Traffic: mitmproxy Patterns

Reverse mode for TCP/TLS forwarding:

```bash
mitmdump --mode reverse:targethost:3888 --ssl-insecure
```

SOCKS mode for multi-host traffic:

```bash
mitmproxy --mode socks5 --listen-port 8081 -k
```

Combined client launch:

```bash
java -Dhttp.proxyHost=localhost -Dhttp.proxyPort=8080 \
     -Dhttps.proxyHost=localhost -Dhttps.proxyPort=8080 \
     -DsocksProxyHost=localhost -DsocksProxyPort=8081 \
     -Djavax.net.ssl.trustStore=cacerts \
     -Djavax.net.ssl.trustStorePassword=changeit \
     -jar app.jar
```

### Binary Payload Triage via Magic Bytes

The TCP or HTTP payload may appear non-human-readable. Look out for the following 'magic bytes' to identify the content of the traffic.

| Magic bytes | Likely type | What to do |
| --- | --- | --- |
| `1F 8B` | gzip | Decompress before analysis |
| `AC ED` | Java serialization | Evaluate deserialization surface |
| `4A 52 4D 49` | JRMI | Treat as RMI-related transport |
| `78` | deflate | Decompress and retry parsing |
| `30 82` | DER ASN.1 | Usually certificate/key material |

### Java TLS-Specific Checks

During static and runtime analysis, explicitly search for:

- custom `X509TrustManager` or `X509ExtendedTrustManager`,
- certificate pinning logic,
- alternate SSL contexts for selected endpoints,
- permissive trust bypasses in debug modes.

Verbose TLS diagnostics:

```bash
java -Djavax.net.debug=all -jar app.jar
```

---

## 2) Decompilation and Static Analysis

### Primary Path

Start with jd-gui:

```bash
jd-gui target.jar
```

Then export sources and open in IntelliJ for cross-file navigation.

### Alternative Decompilers

```bash
java -jar cfr.jar target.jar --outputdir ./decompiled
java -jar procyon-decompiler.jar target.jar -o ./decompiled
```

Different decompilers recover different structures from difficult bytecode.  
For critical methods, compare outputs across tools.

### Static Recon Priorities

Locate and map:

1. startup entrypoint and launch flow,
2. endpoint discovery and configuration loading,
3. authentication/session code paths,
4. trust and TLS handling,
5. serialization and protocol framing logic.

### Obfuscation Strategy

Common signs:

- unreadable symbol names,
- encrypted string helpers,
- heavy reflection,
- unusual control flow noise.

Effective approach:

1. anchor analysis by imports and constants,
2. identify string decryptor functions,
3. extract runtime values via debugger breakpoints,
4. rebuild intent from runtime call traces.

### Reflection and Classloading Pitfalls

If static mapping looks incomplete, check for:

- `Class.forName`,
- `Method.invoke`,
- dynamic proxies,
- custom classloaders,
- runtime-downloaded JARs from JNLP resources.

---

## 3) Runtime Debugging (JDWP + IntelliJ)

### JDWP Basics

Enable remote debugging with:

```text
-agentlib:jdwp=transport=dt_socket,server=y,suspend=n,address=5005
```

Flag meanings:

| Flag | Meaning |
| --- | --- |
| `transport=dt_socket` | Socket transport for debugger communication |
| `server=y` | Target JVM listens for debugger attach |
| `suspend=n` | App starts immediately without waiting |
| `address=5005` | Listen port |

### Standalone Launch with Debug + Proxy

```bash
java -agentlib:jdwp=transport=dt_socket,server=y,suspend=n,address=5005 \
     -Dhttp.proxyHost=localhost -Dhttp.proxyPort=8080 \
     -Dhttps.proxyHost=localhost -Dhttps.proxyPort=8080 \
     -Djavax.net.ssl.trustStore=cacerts \
     -Djavax.net.ssl.trustStorePassword=changeit \
     -jar app.jar
```

### JNLP Launch with Debug

```bash
# Windows
set JAVAWS_VM_ARGS=-agentlib:jdwp=transport=dt_socket,server=y,suspend=n,address=5005 -Dhttp.proxyHost=localhost -Dhttp.proxyPort=8080 -Dhttps.proxyHost=localhost -Dhttps.proxyPort=8080

# Linux
export JAVAWS_VM_ARGS="-agentlib:jdwp=transport=dt_socket,server=y,suspend=n,address=5005 -Dhttp.proxyHost=localhost -Dhttp.proxyPort=8080 -Dhttps.proxyHost=localhost -Dhttps.proxyPort=8080"
```

Optional verbosity:

```bash
javaws -verbose -wait app.jnlp
```

### IntelliJ Attach Steps

1. Add decompiled sources to a project.
2. Create a **Remote JVM Debug** configuration.
3. Set host `localhost` and matching port.
4. Start client and then attach debugger.
5. Place method breakpoints on network-relevant methods.

### Runtime Inspection Tactics

- Use conditional breakpoints to reduce loop noise.
- Inspect method args before transport boundary.
- Inspect transformed objects after response parsing.
- Evaluate expressions for trust/proxy/runtime state.

### JDWP Troubleshooting

| Symptom | Likely cause | Resolution |
| --- | --- | --- |
| Attach fails | Wrong process/port | Verify launch args and target PID |
| Source mismatch | Decompiled packages differ | Re-export/decompile and realign structure |
| Early crash | Wrapper behavior | Attach to actual child JVM |
| Restricted introspection | Policy/modules | Add test policy or `--add-opens` |

---

## 4) Runtime Manipulation with Java Agents

### When to Use It

Move to Java Agents when:

- static + debugger analysis still leaves blind spots,
- you need deterministic method-level telemetry,
- temporary runtime bypasses are required in authorized tests.

### ByteBuddy Hook Example

```java
import net.bytebuddy.agent.builder.AgentBuilder;
import net.bytebuddy.implementation.MethodDelegation;
import net.bytebuddy.matcher.ElementMatchers;
import java.lang.instrument.Instrumentation;
import java.lang.reflect.Method;
import java.util.Arrays;
import java.util.concurrent.Callable;
import static net.bytebuddy.implementation.bind.annotation.AllArguments;
import static net.bytebuddy.implementation.bind.annotation.Origin;
import static net.bytebuddy.implementation.bind.annotation.SuperCall;

public class LoggingAgent {
    public static void premain(String args, Instrumentation inst) {
        new AgentBuilder.Default()
            .type(ElementMatchers.named("com.target.CommunicationClient"))
            .transform((builder, td, cl, m, pd) ->
                builder.method(ElementMatchers.named("sendRequest"))
                    .intercept(MethodDelegation.to(new Interceptor())))
            .installOn(inst);
    }

    public static class Interceptor {
        public Object intercept(@Origin Method method,
                                @AllArguments Object[] args,
                                @SuperCall Callable<?> callable) throws Exception {
            System.out.println("[HOOK] " + method.getName() + " args=" + Arrays.toString(args));
            return callable.call();
        }
    }
}
```

Launch command:

```bash
java -javaagent:logging-agent.jar -jar app.jar
```

### Common Agent Targets

- custom trust/pinning verification methods,
- client-side feature/license gates,
- crypto boundary methods (pre/post encryption),
- serialization boundary methods.

Keep these manipulations test-local and reproducible.

---

## 5) Anti-Analysis Overview

| Technique | Detection hint | First response |
| --- | --- | --- |
| TLS pinning | custom trust manager + fingerprint checks | Instrument trust path |
| Anti-debug checks | checks for JDWP or debug flags | Break/hook check logic |
| Integrity checks | startup hash/signature verification | Trace and isolate verification routines |
| Classloader indirection | encrypted or dynamic class loading | Debug loader path and dump loaded classes |

---

## 6) Troubleshooting Matrix

| Symptom | Cause | Fix |
| --- | --- | --- |
| `PKIX path building failed` | Wrong truststore path or missing CA | Import CA into active truststore |
| No proxy hits | Client bypasses JVM proxy args | Use system proxy fallback / inspect proxy selector |
| Binary-only payloads | Non-HTTP protocol | Switch to debugger/protocol-aware flow |
| Attach unstable | Wrong process or wrapper indirection | Attach to effective JVM PID |
| Inconsistent behavior | Dynamic config/class loading | Capture startup artifacts and loaded modules |

## 7) References

- [Oracle JSSE Reference Guide](https://docs.oracle.com/en/java/javase/11/security/java-secure-socket-extension-jsse-reference-guide.html)
- [Oracle JDWP Connection and Invocation Details](https://docs.oracle.com/en/java/javase/11/docs/specs/jpda/conninv.html)
- [Javassist Tutorial](https://www.javassist.org/tutorial/tutorial.html)
- [OpenWebStart](https://openwebstart.com/)
- [List of file signatures](https://en.wikipedia.org/wiki/List_of_file_signatures)
