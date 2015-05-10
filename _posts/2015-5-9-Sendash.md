#My Hobby Project Sendash

Sendash is an application to monitor a network's health.  This includes everything from change management to verifying status checks of VMWare, Palo Alto, network config, etc.  The goals are:
store the data that [vcheck](http://www.virtu-al.net/vcheck-pluginsheaders/vcheck/)  sends as a daily email, display the info and changes
updates if something fails to check in
Install automatically on the hardware devices and update when necessary
Our use case is a group of consultants that receive a very long set of emails every day on the health of each customer's network.  This application will solve the problem of allowing the consultant to see what has changed at each site and if any site fails to check in to the new application.  The install scripts to create the health check are going to be configured by a web service.  These scripts will live on the above mentioned hardware and are required to not install anything such as Python, BASH, git, and anything useful (/wink), and since they are Windows boxes will be written in PowerShell.  

The web service will have a CRUD frontend to allow users to force the PowerShell scripts to update, to add clients, and to add application users.  Endpoints add themselves through an API call, but go into a pending status.  Users with the correct roles will be able to approve the new endpoint.  The service will also listen to a GitHub hook so that if any of the scripts change, it will force all endpoints that use that script to update.  So far this description would be great for a Spring ROO application, but as we started hashing out the details for the next steps I changed my mind.  

The second phase is an interactive front-end (I see it as a fancy single page reactive application – just to  throw some buzz words out there)  which derives it's data from health check scripts that were installed in the first phase.  Let's say we have a NetApp in our network.  We can collect its name, state, number of disks, raid type, raid size, mirror size, volume count, total size in GB, used size in GB, available size in GB, and the percentage used.  Then you also have a LUN on a Storage Area Network (SAN) on the same network that has a name, creation date, and the size.  The PowerShell scripts that were installed in the first phase are configured to send in health checks with the mentioned data.  This data collected will be able to change at any given time based on the consultants' needs by updating one of the scripts and pushing to github.  This means the app needs to be able to handle the modifications without code changing, the application building, and redeploying.  The front-end initially displays all the data collected to the user and denotes changes, such as the number of disks changing from 4 to 2.  Each user can configure how they see the data, one may not want to see the SAN's info at all, another may only want to see the available size in GB on the Netapp.  In other words entire tables or columns of the data can be hidden on a per person basis.  

Above I mentioned the major system requirements of this hobby project; next are the team requirements or our soft skill evaluation of our combined technical knowledge.  We have two people working on the powershell scripts, one of which will help me with the GUI.  Both are networking consultants, not programmers.  I myself prefer back-end work to front-end, but have always had to do front-end dev so I can do both.  Here I have to choose the right tools for the job, but also the right tools for the team.  My colleague knows HTML and can hack away at JavaScript until he understands what is going on, but has never programmed in Java.  He has minimal C# exposure from writing in powershell for so long.  I have decided to go with a complete RESTful web service where the app makes web service calls to remove any server side code in the GUI.  This group of API calls will be separate microservices.  This means he can jump in whenever necessary without having to learn another language.

Since I haven't created a project in a couple years I looked at two options:  Spring Roo and Spring Boot.  I used Spring Roo when I originally started this project, but dropped it from lack of consultant support.  

##Spring Roo
Spring Roo aims to, "fundamentally improve Java developer productivity without compromising engineering integrity or flexibility" http://en.wikipedia.org/wiki/Spring_Roo.  Spring Roo not only configures your Spring app, but generates code. The auto-configuration is great; if you have ever started a Spring application manually from scratch -  it's painful.  The worst part is getting all the configuration right in the beginning.  Between the size of Spring and how many tutorials there are for previous versions where one thing may be causing the whole thing to spew the same stacktrace over and over I am grateful to hand this job off to a CLI.

The 1.1.0.Release of Roo uses an OSGi container.  This allows users to install/uninstall, start and stop addons while working in the Roo shell.  OSGi was exciting a couple years ago and although I only wrote a simple Roo plugin for adding Twitter's Bootstrap, I really didn't do much with it.  I did have to add a spring security addon.  

The best part of Spring Roo for my application:
* It setup all my Spring config for me
* Integration tests & Selenium tests were started
* The CRUDS for my first couple of domain objects were quickly created
* Aspect J and non-anemic domains

The worst part of Spring Roo for my application:
* After the initial setup and my first two domain objects I stopped using the code generation
* I tore out tiles
* The default CSS and GUI is lacking

##Spring Boot

Directly from their website in bold letters:  “Absolutely no code generation and no requirement for XML configuration.” http://projects.spring.io/spring-boot/ Both Spring Boot and Spring Roo favor convention over configuration, but the Spring Roo configuration was something I was used to from previous projects.  The Spring Boot convention uses annotations to configure the app and although I do prefer annotations over XML I feel like there is a big black curtain blocking what is going on until I research more.  

To get up and running without spending hours reading the docs I cloned a sample project (https://github.com/khoubyari/spring-boot-rest-example) and started converting it to my own.  The Spring Boot site has plenty of sample projects, but many of them focus on one feature:  JPA, or MVC, or consuming a RESTful service (http://spring.io/guides).  The nice thing about the project mentioned above is that it covers serving a RESTful web service, Spring Data with JPA, CRUD functionality, and a health check + metrics. Some issues with this is that it was created six months ago and already out of date.  I updated the boot version and the yaml file broke.  I sent a pull request with the updates to the project owner to help the next person down the line.  I appreciated this sample because it was one of my first gotchas when I was writing my own from a tutorial, but doing some things “my way”, was that I put the Application.java (my configuration file and main method) in a config directory on the same level as my API files.  My first app never worked.  I later realized that the file needed to be at the same level or above the files that need to run it.   


The best part about Spring Boot for my application:
* It has no say in how my Java code is written (no code generation)
* The embedded tomcat for running locally
* It configures the application how I expect it to be configured
* No XML to configure (although I do have a pom.xml that may get converted to a gradle build at some point)
* I'm not boxed into one testing framework without ripping out the old one.  I can try Spock this go round.


The worst part about Spring Boot for my application is all of my unknowns:
* The guides were very simple in scope.  Once I started looking at samples on GitHub using REST with JPA and some metrics I was a lot happier.  My biggest questions were setting it up, such as do we do all the configuration in one Application.java file or one for each item configured?  How is the directory structure to be laid out?
* There are a lot of configuration annotations that cause a little worry.  Do I need to have it set up correctly on day one?
* I'm not sure if Spring Boot is production ready.  I've seen several articles dated 2014 that it wasn't, but my application is not the next Amazon nor Facebook.  It's a B2B service for less than 1,000 users starting out.

