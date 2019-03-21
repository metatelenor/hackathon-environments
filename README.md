#What did I want?
I am tired of not having a good way for others to look at how my webapp looks like when doing code reviews and having to share the test/staging environments with others when I just want to verify that my webapp works like it is supposed to outside of my localhost. I therefore wanted a way of creating a test environment for every branch someone makes on git so that everyone can have their own environment to fool around with for the feature that they are working on.

#What did I do?
After googling around for a while I found this article https://jenkins.io/blog/2015/12/03/pipeline-as-code-with-multibranch-workflows-in-jenkins/ that came with a docker image for running jenkins with a local git repository and a sample pipeline for building a web-app.

I then ran the docker image as instructed in the article: `docker run -p 8080:8080 -p 8081:8081 -p 9418:9418 -ti jenkinsci/workflow-demo`. I did however see a lot of error message, but as a proper developer I quickly opened a new terminal and ignored the output from the docker container, praying that it would still work (and it did)

Running this docker image hosts a jenkins build server on your computer and allows you to access the jenkins UI on http://localhost:8080

It also hosts a git repository that you can clone using `git clone git://localhost/repo` with a sample pipeline for hosting a sample webapp.
So I cloned the git repository and then I was done.... or was I?

#What changes did I have to make
The first thing I noticed when running the docker container was that it was configured so that it would create a job for every branch in the project, but it only actually built the webapp if it was on the master branch, which meant that you had to trigger the new jobs manually in order to deploy them. So the first thing I did was remove this configuration:

* http://localhost:8080/job/cd/configure
* Remove everything listed under exceptions
* Save
* ?????
* PROFIT

I then realized that the pipeline(contained in the Jenkinsfile) would deploy every branch to localhost:8081/staging which I did not want, so I made some changes to the Jenkinsfile. It would now deploy every branch to localhost:8081/<YourBranchNameHere> and on the master branch would ask you before deploying to staging and production, not allowing you to deploy to these environments unless you were on deploying the what is on the master branch. You can find the new Jenkinsfile that I made in this git repository

#How would you have done this without the docker image supplied from the article
* Download jenkins and run in locally
* Create a git repository somewhere
* Create a jenkins multibranch pipeline job
* Configure the job to use the git repository
* Create a sample webapp using whatever framework and add it to get git repository
* Create a jenkins pipeline that builds and runs the webserver, hosting the webserver on your localhost using the branchname as part of the url, alternatively push it to AWS somehow 

#What did you not have the time to do
The webservers never die! When a branch is deleted the job dies, but the webservers do not. I would have wanted to create a jenkins job that kills webservers that do not have a correpsonding living branch. It could also delete webservers that have been running for more than say 1 week so that we don't keep things running on abandoned branches. If I was to host the things on AWS it would have been even more important to make sure that you kill webservers dead branches as soon as possible
