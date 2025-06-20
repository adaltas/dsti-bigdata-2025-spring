# Big Data Ecosystem

## Lab 2: HDFS and YARN interaction

### Goals

- Get familiar with the CLIs
- Copy files to HDFS
- Submit an application in YARN
- Track an application using the YARN Web UI

**Before you start**

As a member of a group, you have permission to work within resources named after your group name.
To make it easier to reference your group name in scripts and commands, you can create an environment variable called GROUP using this script:

```bash
echo "export GROUP=<group>" >> ~/.bashrc
source .bashrc
```

After running this script, you can use the variables $USER and $GROUP to reference your user and group name respectively in your scripts and commands.

### Copy files to HDFS

Using the official [HDFS DFS Commands Guide](https://hadoop.apache.org/docs/current/hadoop-project-dist/hadoop-common/FileSystemShell.html):

1. Create a directory named after your `$USER` in your `$GROUP` HDFS directory (`/education/$GROUP/$USER`)
2. Create a subdirectory `lab2` in the directory created in 1.
3. Create a file named `sentence.txt` on the local file system (`/home/$USER/lab1`) and write a sentence inside that file
4. Copy the file to your `lab2` HDFS directory
5. Get information about the blocks composing your file, their replica and locations using `hdfs fsck <path-to-your-file>`

### Submit an application in YARN

To test YARN, we will run the Pi example from the MapReduce example JAR. This is a MapReduce program that computes Pi using the [Quasi-Monte Carlo method](https://en.wikipedia.org/wiki/Quasi-Monte_Carlo_method) (1st argument = number of mappers, 2nd argument = number of samples).

1. Run the command:

   ```bash
   yarn jar /usr/hdp/3.1.0.0-78/hadoop-mapreduce/hadoop-mapreduce-examples-3.1.1.3.1.0.0-78.jar pi 6 100000000
   ```

2. Wait until you find the result
3. Run the command `yarn app -list` that shows your apps running in YARN

### WordCount example application

The WordCount example is also located in the MapReduce example JAR. It takes several arguments:

- 1 or more input directory
- 1 output directory

1. Look at the content of the input directory: `/education/$GROUP/$USER/lab2/`
2. Run the command:

   ```bash
   yarn jar /usr/hdp/current/hadoop-mapreduce-client/hadoop-mapreduce-examples-3.1.1.3.1.0.0-78.jar \
    wordcount /education/$GROUP/$USER/lab2 \
    /education/$GROUP/$USER/lab2/output-moby-dick
   ```

3. Check out the output directory: `/education/$GROUP/$USER/lab2/output-moby-dick`

---

> NOTE: The following session is not mandantory and will be time-consuming due to the Kerberos configuration. Make sure to begin only if you have completed the above tasks.

### Setting up password-less SSH

Follow tutorial: [How to Setup Passwordless SSH Login](https://linuxize.com/post/how-to-setup-passwordless-ssh-login/)  
**NOTE:** Setting up passwordless ssh will bypass the automatic kerberos ticket renewal that happens when using a password to log in. If you receive an error when interacting with the cluster that involves `[TOKEN,KERBEROS]` or any other error mentioning kerberos, first try renewing your ticket on the edge node.

```bash
# Use kinit without flags to update ticket
kinit
# #> denotes standard output
#> Password for <user>:<enter-password-here>
```

### Accessing Hadoop Web UIs using Kerberos

To see the Hadoop Web UIs, you will need to configure your computer. The Kerberos protocol is used to authenticate.

The configuration depends on your OS:

- On Linux and MacOS

  - Nothing to install
  - Create the file `/etc/krb5.conf` on your computer with the following content:

    ```ini
    [libdefaults]
     default_ccache_name = FILE:/tmp/krb5cc_%{uid}
     default_realm = AU.ADALTAS.CLOUD
     dns_lookup_realm = false
     dns_lookup_kdc = true
     rdns = false
     ticket_lifetime = 24h
     forwardable = true
     udp_preference_limit = 0

    [realms]
     AU.ADALTAS.CLOUD = {
      kdc = ipa1.au.adaltas.cloud:88
      master_kdc = ipa1.au.adaltas.cloud:88
      admin_server = ipa1.au.adaltas.cloud:749
      default_domain = au.adaltas.cloud
     }

    [domain_realm]
     .au.adaltas.cloud = AU.ADALTAS.CLOUD
     au.adaltas.cloud = AU.ADALTAS.CLOUD
     ipa1.au.adaltas.cloud = AU.ADALTAS.CLOUD
    ```

  - After that you can get a Kerberos ticket using `kinit $USER` (same user name that the one used to connect with SSH)
  - Check your ticket with `klist`
  - Add the following properties in your Firefox `about:config` page:

    ```ini
    network.negotiate-auth.delegation-uris = .au.adaltas.cloud
    network.negotiate-auth.trusted-uris = .au.adaltas.cloud
    ```

- On Windows, the installation is more tricky. Follow this article: [Kerberos and Spnego authentication on Windows with Firefox](https://www.adaltas.com/en/2019/11/04/windows-krb5-client-spnego/). **NOTE: when interacting with the MIT-Kerberos gui, the value of your username is `<username>@AU.ADALTAS.CLOUD`, and the value of your password is the one you used to connect with SSH**

Once your computer and your Firefox browser are configured and you have a Kerberos ticket:

1. Open the page http://yarn-rm-1.au.adaltas.cloud:8088/ui2/#/yarn-apps/apps
2. Run the Pi example once again
3. Locate your application in the UI and check resource usage
4. Rerun the example several times and see the locations of the containers changing
