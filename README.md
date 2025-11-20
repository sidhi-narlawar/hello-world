Detailed step-by-step (mapped to the Task you posted)

Maven Java Automation (Project flow: MavenJava_Build → MavenJava_Test → pipeline view)

Step A — Create MavenJava_Build
	1.	In Jenkins dashboard click New Item.
	2.	Type name: MavenJava_Build. Select Freestyle project. Click OK.
	3.	Description box: enter Java Build demo.
	4.	Source Code Management → Select Git → Repository URL:
https://github.com/bhavagna06/Maven.git
Branches to build: */Main or */master (pick whichever branch the repo uses).
	5.	Build → Add build step Invoke top-level Maven targets
	•	Maven version: MAVEN_HOME (the name you configured)
	•	Goals: clean
	6.	Add another Invoke top-level Maven targets build step
	•	Maven version: MAVEN_HOME
	•	Goals: install (this builds and produces the artifact, e.g., .jar)
	7.	Post-build Actions → Archive the artifacts → Files to archive: **/*
(This archives all files produced so downstream jobs can copy them.)
	8.	Post-build Actions → Build other projects → Projects to build: MavenJava_Test
Trigger: Only if build is stable (so downstream runs only on success).
	9.	Apply and Save.

Expected outcome: this job clones the repo, runs mvn clean then mvn install, archives artifacts (target folder), and triggers MavenJava_Test on success.

⸻

Step B — Create MavenJava_Test
	1.	New Item → MavenJava_Test → Freestyle → OK.
	2.	Description: Test demo.
	3.	Build Environment: check Delete the workspace before build starts (keeps workspace clean).
	4.	Add build step Copy artifacts from another project
	•	Project name: MavenJava_Build
	•	Which build: Stable build only
	•	Artifacts to copy: **/* (copies everything archived)
	5.	Add build step Invoke top-level Maven targets
	•	Maven version: MAVEN_HOME
	•	Goals: test (runs unit tests)
	6.	Post-build Actions → Archive the artifacts → Files to archive: **/*
	7.	Apply and Save.

Expected outcome: copies build outputs from the build job, runs tests, archives test artifacts/logs.

⸻

Step C — Create Pipeline View
	1.	In Jenkins dashboard, click the + beside All (or New View).
	2.	Enter name: Java_Pipeline. Select Build pipeline view. Click OK.
	3.	Pipeline settings: Layout: Based on upstream/downstream relationship. Initial job: MavenJava_Build.
	4.	Apply and Save.

Expected outcome: visual flow showing MavenJava_Build → MavenJava_Test. You can click builds and view console output.

⸻

Step D — Run and verify
	1.	In pipeline view, click the trigger (or click Build Now on MavenJava_Build).
	2.	For each job, click the small black console icon (or open the build and click Console Output) to see logs. Look for:
	•	Maven goals executed (clean → install → test).
	•	Build status SUCCESS (green).
	•	Tests passed (or failing tests reported).
	3.	Confirm artifacts are archived (look in job → Build # → Artifacts or Workspace if needed).

⸻

Maven Web Automation (flow: MavenWeb_Build → MavenWeb_Test → MavenWeb_Deploy → pipeline view)

Step A — Create MavenWeb_Build

Same steps as Java build but repo and name differ:
	•	Repo: https://github.com/bhavagna06/maven-web-app.git
	•	Jobs: MavenWeb_Build, goals: clean and install
	•	Archive artifacts **/*
	•	Post-build: Build other projects → MavenWeb_Test (Only if build is stable)

Step B — Create MavenWeb_Test
	•	Name: MavenWeb_Test
	•	Delete workspace before build starts.
	•	Copy artifacts from MavenWeb_Build (Stable build only).
	•	Invoke Maven goals: test
	•	Archive artifacts **/*
	•	Post-build action: Build other projects → MavenWeb_Deploy

Step C — Create MavenWeb_Deploy
	•	Name: MavenWeb_Deploy
	•	Description: Web Code Deployment
	•	Delete workspace before build starts.
	•	Copy artifacts from MavenWeb_Test.
	•	Post-build Actions → Deploy WAR/EAR to a container:
	•	WAR/EAR File: **/*.war (the artifact produced by mvn package / install)
	•	Context path: Webpath (this will cause app to be reachable at http(s)://<tomcat-host>:<port>/Webpath)
	•	Add container: Tomcat 9.x remote
	•	Credentials: choose stored credentials (username & password for Tomcat manager). The task shows admin / 1234 — store these in Jenkins Credentials, but use secure credentials in real life.
	•	Tomcat URL: https://localhost:8080/
	•	Apply and Save.

Expected outcome: when triggered, this job copies the WAR from earlier steps and deploys it via Tomcat manager to https://localhost:8080/Webpath.

⸻

Step D — Create Pipeline View for MavenWeb
	•	Create view named MavenWeb_Pipeline, initial job MavenWeb_Build, layout based on upstream/downstream relationship. Save.

Step E — Run pipeline & verify deploy
	1.	Run pipeline (click Run on the pipeline view or Build Now MavenWeb_Build).
	2.	Watch console outputs in each job. Confirm each is SUCCESS (green).
	3.	After MavenWeb_Deploy completes, open Tomcat manager (or http(s)://localhost:8080/), find context /Webpath in manager app and click it — the web app should open.
	4.	If errors appear at deploy step, check Tomcat logs (catalina.out) and Jenkins console for deployment errors (bad credentials, incompatible WAR, wrong context path, SSL cert issues with https).




