Functional tests
================

Described are test for availability and functionality of the GNUU core systems:

1. Set the newsfeed to "*" to receive all available newsgroups and fill up the outgoing queue for your site
2. Set batchtime to 300 seconds
3. Set compression to *gzip*
4. Wait 5-10 min so some data are stored
5. Call the uucp system with your client. You should receive gzipped uucp packages
6. Repeat steps 3-5 with compression "bzip2", "szip", "compress"
7. Produce a news article for newsgroup de.test, call your local batcher and send the uucp package to the server. In result you should see after some minutes the article [here](https://groups.google.com/g/de.test)
8. Produce an e-mail to an outside recipient, call your local batcher and send the uucp package to the server. The e-mail should delivered after some minutes. Step 7+8 are done with your favvorite compressor. No full test is required.
