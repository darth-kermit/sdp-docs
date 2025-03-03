.. _JTE Advanced Features MultiMethod Steps: 

------------------
Multi-Method Steps
------------------

While learning about Pipeline Lifecycle Hooks, we created a step that: 

* implemented multiple methods 
* did not implement a ``call`` method 

In this section, we're going to dive into multi-method steps in a little 
more detail. 

.. important:: 

    Have you ever wondered why library steps create a method named ``call``? 

    This is because, in groovy, ``something()`` gets translated to ``something.call()``. 

If we understand this concept, then it would make sense that we could define other methods 
within our steps and invoke them by their full name. 

==============================
When to use Multi-Method Steps
==============================

The most common use case for defining multiple methods inside one step file is when you're 
creating some utility functionality. 

To demonstrate this, let's create a mock ``git`` utility that can ``add``, ``commit``, and
``push``. 

======================
Create the Git Library
======================

In the same Pipeline Configuration Repository we used for JTE: The Basics, create a 
``git`` library. 

Because we're creating a git utility, add a file called ``git.groovy`` with the contents: 

.. code:: groovy 

    /*
        takes an arraylist of files to pass to git add 
    */
    void add(ArrayList files){
        println "git add ${files.join(" ")}"
    }

    /*
        takes a string commit message to pass to git commit 
    */
    void commit(String message){
        println "git commit -m ${message}" 
    }

    /*
        performs the git push
    */
    void push(){
        println "git push" 
    }

In this example, we're creating a step that serves as a utility wrapper.  These are typically **not** 
invoked directly by Pipeline Templates but rather consumed by other steps.  That is why it is okay,
in this case, to accept input parameters for these methods. 

We will be invoking this functionality directly from the Pipeline Template to demonstrate its functionality. 

.. important:: 

    The filestructure for your Pipeline Configuration libraries directory should 
    now be: 

    .. code:: 

        .
        ├── libraries
            ├── ansible
            │   └── deploy_to.groovy
            ├── git
            │   └── git.groovy
            ├── gradle
            │   └── build.groovy
            ├── maven
            │   └── build.groovy
            ├── sonarqube
            │   └── static_code_analysis.groovy
            └── splunk
                ├── splunk_pipeline_end.groovy
                ├── splunk_pipeline_start.groovy
                └── splunk_step_watcher.groovy

=================================
Update the Pipeline Configuration
=================================

Update the Pipeline Configuration to load the ``git`` library. 

The ``libraries`` portion of the Pipeline Configuration should now be: 

.. code:: groovy 

    libraries{
        maven
        sonarqube
        ansible
        splunk{
            afterSteps = [ "static_code_analysis", "unit_test" ]
        }
        git
    }

=======================
Use the new Git Utility
======================= 

Prepend to the existing Pipeline Template: 

.. code:: groovy 

    git.add(["a", "b", "c"])
    git.commit "my commit message" 
    git.push()

.. important:: 

    When invoking a non-call method defined within a step, you do so 
    by ``<step_name>.<method_name>(<arguments>)``. 

================
Run the Pipeline
================ 

Run the pipeline again and you will see logs similar to: 

.. code-block:: text 

    Started by user admin
    Running in Durability level: MAX_SURVIVABILITY
    [Pipeline] Start of Pipeline
    [JTE] Pipeline Configuration Modifications (show)
    [JTE] Loading Library maven (show)
    [JTE] Library maven does not have a configuration file.
    [JTE] Loading Library sonarqube (show)
    [JTE] Library sonarqube does not have a configuration file.
    [JTE] Loading Library ansible (show)
    [JTE] Library ansible does not have a configuration file.
    [JTE] Loading Library splunk (show)
    [JTE] Library splunk does not have a configuration file.
    [JTE] Loading Library git (show)
    [JTE] Library git does not have a configuration file.
    [JTE] Creating step unit_test from the default step implementation.
    [JTE] Obtained Pipeline Template from job configuration
    [Pipeline] node
    Running on Jenkins in /var/jenkins_home/workspace/single-job
    [Pipeline] {
    [Pipeline] writeFile
    [Pipeline] archiveArtifacts
    Archiving artifacts
    [Pipeline] }
    [Pipeline] // node
    [JTE] [@Init - splunk/splunk_pipeline_start.call]
    [Pipeline] echo
    Sending Splunk event for beginning of the pipeline!
    [JTE] [@BeforeStep - splunk/splunk_step_watcher.before]
    [Pipeline] echo
    Splunk: running before the git library's git step
    [JTE] [Step - git/git.add(ArrayList)]
    [Pipeline] echo
    git add a b c
    [JTE] [@BeforeStep - splunk/splunk_step_watcher.before]
    [Pipeline] echo
    Splunk: running before the git library's git step
    [JTE] [Step - git/git.commit(String)]
    [Pipeline] echo
    git commit -m my commit message
    [JTE] [@BeforeStep - splunk/splunk_step_watcher.before]
    [Pipeline] echo
    Splunk: running before the git library's git step
    [JTE] [Step - git/git.push()]
    [Pipeline] echo
    git push
    [JTE] [Stage - continuous_integration]
    [JTE] [@BeforeStep - splunk/splunk_step_watcher.before]
    [Pipeline] echo
    Splunk: running before the Default Step Implementation library's unit_test step
    [JTE] [Step - Default Step Implementation/unit_test.call()]
    [Pipeline] stage
    [Pipeline] { (Unit Test)
    [Pipeline] node
    Running on Jenkins in /var/jenkins_home/workspace/single-job
    [Pipeline] {
    [Pipeline] isUnix
    [Pipeline] sh
    + docker inspect -f . maven
    .
    [Pipeline] withDockerContainer
    Jenkins seems to be running inside container cc7140d4fb91bef940e2fabe7225dcbcc9b44a3a5e17ee703b8fcbe42e53a17c
    $ docker run -t -d -u 0:0 -w /var/jenkins_home/workspace/single-job --volumes-from cc7140d4fb91bef940e2fabe7225dcbcc9b44a3a5e17ee703b8fcbe42e53a17c -e ******** -e ******** -e ******** -e ******** -e ******** -e ******** -e ******** -e ******** -e ******** -e ******** -e ******** -e ******** -e ******** -e ******** -e ******** -e ******** -e ******** -e ******** -e ******** -e ******** -e ******** -e ******** -e ******** maven cat
    $ docker top 4bedf0c251a44759408b76ac7dc6db2bebef8438f95018911a0424dfeb68f18d -eo pid,comm
    [Pipeline] {
    [Pipeline] unstash
    [Pipeline] sh
    + mvn -v
    Apache Maven 3.6.2 (40f52333136460af0dc0d7232c0dc0bcf0d9e117; 2019-08-27T15:06:16Z)
    Maven home: /usr/share/maven
    Java version: 11.0.5, vendor: Oracle Corporation, runtime: /usr/local/openjdk-11
    Default locale: en, platform encoding: UTF-8
    OS name: "linux", version: "4.9.125-linuxkit", arch: "amd64", family: "unix"
    [Pipeline] }
    $ docker stop --time=1 4bedf0c251a44759408b76ac7dc6db2bebef8438f95018911a0424dfeb68f18d
    $ docker rm -f 4bedf0c251a44759408b76ac7dc6db2bebef8438f95018911a0424dfeb68f18d
    [Pipeline] // withDockerContainer
    [Pipeline] }
    [Pipeline] // node
    [Pipeline] }
    [Pipeline] // stage
    [JTE] [@AfterStep - splunk/splunk_step_watcher.after]
    [Pipeline] echo
    Splunk: running after the Default Step Implementation library's unit_test step
    [JTE] [@BeforeStep - splunk/splunk_step_watcher.before]
    [Pipeline] echo
    Splunk: running before the maven library's build step
    [JTE] [Step - maven/build.call()]
    [Pipeline] stage
    [Pipeline] { (Maven: Build)
    [Pipeline] echo
    build from the maven library
    [Pipeline] }
    [Pipeline] // stage
    [JTE] [@AfterStep - splunk/splunk_step_watcher.after]
    [Pipeline] echo
    Splunk: running after the maven library's build step
    [JTE] [@BeforeStep - splunk/splunk_step_watcher.before]
    [Pipeline] echo
    Splunk: running before the sonarqube library's static_code_analysis step
    [JTE] [Step - sonarqube/static_code_analysis.call()]
    [Pipeline] stage
    [Pipeline] { (SonarQube: Static Code Analysis)
    [Pipeline] echo
    static code analysis from the sonarqube library
    [Pipeline] }
    [Pipeline] // stage
    [JTE] [@BeforeStep - splunk/splunk_step_watcher.before]
    [Pipeline] echo
    Splunk: running before the ansible library's deploy_to step
    [JTE] [Step - ansible/deploy_to.call(ApplicationEnvironment)]
    [Pipeline] stage
    [Pipeline] { (Deploy To: dev)
    [Pipeline] echo
    performing a deployment through ansible..
    [Pipeline] echo
    deploying to 0.0.0.1
    [Pipeline] echo
    deploying to 0.0.0.2
    [Pipeline] }
    [Pipeline] // stage
    [Pipeline] timeout
    Timeout set to expire in 5 min 0 sec
    [Pipeline] {
    [Pipeline] input
    Approve the deployment?
    Proceed or Abort
    Approved by admin
    [Pipeline] }
    [Pipeline] // timeout
    [JTE] [@BeforeStep - splunk/splunk_step_watcher.before]
    [Pipeline] echo
    Splunk: running before the ansible library's deploy_to step
    [JTE] [Step - ansible/deploy_to.call(ApplicationEnvironment)]
    [Pipeline] stage
    [Pipeline] { (Deploy To: Production)
    [Pipeline] echo
    performing a deployment through ansible..
    [Pipeline] echo
    deploying to 0.0.1.1
    [Pipeline] echo
    deploying to 0.0.1.2
    [Pipeline] echo
    deploying to 0.0.1.3
    [Pipeline] echo
    deploying to 0.0.1.4
    [Pipeline] }
    [Pipeline] // stage
    [JTE] [@CleanUp - splunk/splunk_pipeline_end.call]
    [Pipeline] echo
    Splunk: end of the pipeline!
    [Pipeline] End of Pipeline
    Finished: SUCCESS