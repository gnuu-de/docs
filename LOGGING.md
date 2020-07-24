Kubernetes Log Pattern
======================

In GNUU we push "traditional" internet services like email, news and uucp
into container. Kubernetes has a {Logging Concept](https://kubernetes.io/docs/concepts/cluster-administration/logging/)
and [12 Factor App](https://12factor.net) describes: you have ship log to STDERR and STDOUT.
That's atypical for traditionally service which this option only provides for developement
or debugging.

INN
---

The Internet Networt News service version 2 provides this start flags:

```
       -d, -f
           innd normally puts itself into the background, points its standard output and error to log files, and disassociates itself
           from the terminal.  Using -d prevents all of this, resulting in log messages being written to standard output; this is
           generally useful only for debugging.  Using -f prevents the backgrounding and disassociation but still redirects output; it
           may be useful if you want to monitor innd with a program that would be confused by forks.
```

Fine, in the entrypoint.sh we put the normal INN start script /usr/lib/news/bin/rc.news


```
    spec:
      containers:
      - image: gnuu/inn
        imagePullPolicy: Always
        command:
        - /entrypoint.sh

```

Test:


```
$ kubectl -n gnuu logs news-0 --container news

Jul 24 21:14:13.109 + uucp.gnuu.de <rffiv0$lg0$1@dont-email.me> (@05010000194A000000000003D60C00000000@) 2432 6913303049587 6913303497348
Jul 24 21:14:55.895 + uucp.gnuu.de <5f1b4f35$0$541$65785112@news.neostrada.pl> (@050100001B6C0000000000044FF000000000@) 10113 6913303049587 6913305887985
Jul 24 21:15:00.276 + uucp.gnuu.de <rffj0j$19q9$1@neodome.net> (@050100001A89000000000004468E00000000@) 1196 6913303049587 6913303497348
```

That's only incoming article log, normaly pushed to /var/log/news/news

Other messages are send to syslog. That means, we have to install rsyslog in Dockerfile.inn,
prepare the config to prefend kernel log, create and add named pipe and send the logs to 
the pipeline:

```
sed -i '/imklog/s/^module/# /' /etc/rsyslog.conf
echo 'module (load="builtin:ompipe")' >> /etc/rsyslog.conf

mkdir /newslog && mkfifo /newslog/news.fifo && chown syslog:syslog /newslog/news.fifo
echo "*.*                             action(type=\"ompipe\" Pipe=\"/newslog/news.fifo\")" > /etc/rsyslog.d/50-default.conf
```

all other syslog destinations are removed.

To prevent a mix of log formats and services on the same STDOUT
 we use a sidecar and a second container for syslog output:


```
[...]
      initContainers:
      - name: install
        image: gnuu/busybox
        command: ['bash', '-x', "/sidecar/run.sh"]
        volumeMounts:
        - name: newslog
          mountPath: "/newslog"
[...]
      volumes:
      - name: newslog
        emptyDir: {}

```

An empydir volume will create in the [statefulset-deployment](https://github.com/gnuu-de/k8s/blob/master/news/statefulset.yaml)
The volume ist first mounted in sidecar. The run.sh will create the
named pipe /newslog/news.fifo on the temporary volume. 


```
[...]
      containers:
      - image: gnuu/inn
        imagePullPolicy: Always
        command:
        - /entrypoint.sh
[...]
        volumeMounts:
        - name: newslog
          mountPath: "/newslog"

```

The main news container will also mount the /newslog with the generated named pipe.
rsyslogd is starting in entrypoint.sh, logs are delivery to the pipe

The third mount comes from another container


```
[...]
      - image: gnuu/busybox
        imagePullPolicy: Always
        command: ['sh', '-c']
        args: ['tail -n+1 -f /newslog/news.fifo']
        name: newslog
[...]
        volumeMounts:
        - name: newslog
          mountPath: "/newslog"

```

The startup command for this container is tail to the fifo file. The output on
the STDOUT of this container will be the syslog part of INN:


```
2020-07-24T21:44:52.460779+00:00 quickstart1 innd: SERVER starting
2020-07-24T21:44:52.528670+00:00 quickstart1 controlchan[40]: starting
2020-07-24T21:44:54.509889+00:00 quickstart1 innfeed[41]: uucp.gnuu.de:0 cxnsleep no permission to talk: 502 You have no permission to talk.  Goodbye!

```
