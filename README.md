# descartes-issue-generator

This project extends the pitmp-maven-plugin, to generate Gitlab issues out of mutation testing:
to provide that, it adds a new output format, GITLAB-ISSUES, to PIT/Descartes.

The generated Gitlab issues will be overwritten (updated) if the plugin is run more than once,
so you can use it multiple times without generating tons of verbose issues...

## Configuration

To use it, just add some configuration to pitmp-maven-plugin, to declare:
- GITLAB-ISSUES as an output format (requires fullMutationMatrix=true so that list of succeding tests is provided by PIT)
- Gitlab configuration (optional): if you want Gitlab issues to be injected in your repository, specify the destination
(Gitlab url, project and token).

Note that, as fullMutationMatrix=true is required, PIT will only allow XML output in addition to GITLAB-ISSUES
(eg. no HTML nor JSON is allowed). This limitation is due to PIT/Descartes.

Example of Descartes plugin configuration to enable GITLAB-ISSUE output:

```
  <plugin>
    <groupId>eu.stamp-project</groupId>
    <artifactId>pitmp-maven-plugin</artifactId>
    <version>1.3.8-SNAPSHOT</version>
    <!-- All PIT's properties can be used. -->
    <dependencies>
      <dependency>
        <groupId>eu.stamp-project</groupId>
        <artifactId>descartes-issue-generator</artifactId>
        <version>1.0.0-SNAPSHOT</version>
      </dependency>
    </dependencies>
    <!--
      fullMutationMatrix=true required (so that list of succeding tests is provided by PIT) 
      With that option, PIT/Descartes only allows XML output (no HTML, JSON...)
      Output will be written in reportsDirectory (with same default as for Descartes),
      and optionally to Gitlab if appropriate configuration of destination is provided.
    -->
    <configuration>
      <mutationEngine>descartes</mutationEngine>
      <fullMutationMatrix>true</fullMutationMatrix>
      <reportsDirectory>/tmp/pit-reports</reportsDirectory>
      <outputFormats>
        <value>GITLAB-ISSUES</value>
        <value>XML</value>
      </outputFormats>
      <!-- Sample config of Gitlab destination (if provided, Gitlab issues will be generated) -->
      <pluginConfiguration>
        <gitlabUrl>http://localhost</gitlabUrl>
        <gitlabToken>i_rjH-MM3YuFsvAxCTsH</gitlabToken>
        <gitlabProject>2</gitlabProject>
      </pluginConfiguration>
    </configuration>
  </plugin>
```

## Execution

To run Descartes with the configuration above:

```
mvn eu.stamp-project:pitmp-maven-plugin:descartes
```
(Note: generally requires that your project is built before: run "mvn clean install" prior to running Descartes !)

According to issues discovered by Descartes, 3 files may be generated as output:
- git_critical_issue.txt reports code mutations that are not detected by tests at all (GREEN test suite).
- git_minor_issue.txt reports code mutations that SOME tests do not detect.
- git_no_coverage.txt reports missing unit tests.

The 2 first ones are replicated as Gitlab issues (if a proper Gitlab configuration is provided).

## Link with DSpot

If critical issues are detected, the output in git_critical_issue.txt (and the Gitlab issue) suggests DSpot commands (using DSpot maven plugin) to generate tests that may fix the issue.

Example of such output:

```
==========================================================================
CRITICAL TEST FAILURE: test suite GREEN upon code mutation
In class org.objectweb.joram.mom.util.MessageIdListImpl, method get (line 84) was updated as follows:
	All methods instructions replaced by: return null;
	The following test(s) still PASS:
		org.objectweb.joram.mom.proxies.MessageIdListImplEncodingTest.run(org.objectweb.joram.mom.proxies.MessageIdListImplEncodingTest)
==========================================================================
Suggested DSpot fix(es):
mvn eu.stamp-project:dspot-maven:2.1.1-SNAPSHOT:amplify-unit-tests -Dpath-to-properties=/tmp/stamp1379112605526824370.tmp -Dtest=org.objectweb.joram.mom.proxies.MessageIdListImplEncodingTest -Dtest-criterion=JacocoCoverageSelector -Dgenerate-new-test-class=true -Diteration=1
==========================================================================
```
Running the suggested DSpot command, then "mvn test" and Descartes again, may or may not fix the issue.
