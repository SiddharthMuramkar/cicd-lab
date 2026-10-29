Intro
0:00
Hey guys, so AI has changed software development and I think we're at a point where coding and software development
0:05
are no longer synonymous and people are building entire applications with agents without understanding every line of code
0:12
or in some cases any lines of code. And I'm not here to criticize that at least
0:17
in this video. But if you're going to be putting AI generated code into production, which I do all the time, you
0:24
need to have safeguards and you need tests, you need gates, you need a a CI/CD pipeline that catches problems
0:30
before they reach your users. So, in this video, we'll talk about what CI/CD is and actually put it into practice.
0:37
And we're going to be using GitHub actions to automate our checks, deploy to render, and explore features like
0:44
preview deployments. And we'll start by cloning a simple Express and TypeScript app and then set up CI to check our
0:51
types to run our tests and make sure our app builds for production. And to be clear, even if you're vibe coding, this
0:58
is important. In fact, I would say it's even more important. And you don't necessarily need to to do it all
1:03
manually like we're going to do in this video. You can simply tell your agent to add these guard rails. So even if you
1:10
don't understand like the workflow file and and all the steps and everything like that, you can still know the at
1:16
least know the terminology to prompt. So let's get into it. [music]
What Is CI/CD?
1:24
All right, guys. So before anything, I just want to mention we are having kind of a noraster here, which is uh like a
1:30
northeast windstorm if for those of you that aren't from New England. So you might hear some wind, some banging outside. So I apologize for that
1:36
beforehand. Now before we touch any code or get into anything, let's let's break down what CI/CD actually means. So CI
1:44
stands for continuous integration, and it's the process of regularly integrating code changes into a shared
1:50
codebase and automatically checking those changes. And that could include things like type checking, unit tests,
1:57
other types of tests, linting, and just making sure that the application can build. And CD has two common meanings.
2:04
You have continuous delivery and continuous deployment. Now the difference is whether releasing to
2:10
production needs a separate human approval. With continuous delivery, it does need that human. With continuous
2:16
deployment, it happens automatically once those required checks pass. And we're using continuous deployment in
2:23
this project. Now, CI/CD isn't specific to GitHub, even though that's what we'll
2:28
be using. You you can use tools like Jenkins, CircleCI, TravisCCI. These
2:34
tools they automate the the building, testing and deployment of your code and
2:39
they differ in setup and configuration but the basic goal is the same which is to automatically run a series of check
2:46
checks and tasks whenever something changes in your codebase. Now we'll be using GitHub actions because we're
What Is Github Actions?
2:52
already keeping our code on GitHub and we're running our checks right there whenever we make a pull request and it
2:59
can handle deployments as well. But in our setup, GitHub actions will do the CI checks and render will handle the
3:05
building and deploying the application. So where do we where do these checks actually run? So GitHub actions uses
3:12
something called a runner. And in our setup, GitHub gives each job a fresh
3:17
abundu virtual machine. So basically a temporary computer in the cloud. and our workflow, which is a a specific YAML
3:25
file that we're going to create. It tells it to check out the code, set up node, install our dependencies, and run
3:32
our checks. And when the job finishes, that machine, that virtual machine is discarded. So, we're checking that the
3:38
project works in a clean environment without relying on whatever happens to
3:44
be installed on our own machine. And if your tests need a database, you can also have the workflow start up a temporary
3:50
database just for that test run. Our tests don't need one, but we could do that if we wanted to test actual
3:57
database queries. So the workflow that we're building is pretty straightforward. We're going to we'll
4:03
make a change on a separate branch. We'll run our checks locally and open a pull request. GitHub actions will run
4:10
those checks again and we'll configure GitHub to block the merge if they fail. Now once we merge into main render will
4:17
wait for the checks on that commit to pass then build and deploy it. We'll also deliberately break stuff just to
4:23
show that it's working properly. All right, so this is what we're going to do. I have this gist file. I'm going to put this link in the description so you
Clone & Go Over The Project
4:30
can have it. It has all the the commands, all the code, everything you need in order to follow along with this,
4:35
which I would suggest you do because it's going to stick better if you if you follow along. and render has a very
4:41
generous free tier so you can deploy and do everything. So the very first command here is a git clone and you'll have uh a
4:50
link to this CI cd lab starter. This is the repo we're going to use. It's just a
4:55
very simple it's basically just a boilerplate an express typescript app that has a couple routes and it has a
5:02
test. It has everything we need to basically to teach you this this whole process with very minimal code. So,
5:10
let's grab that that I almost said clone. Let's grab that clone and paste
5:15
that in. And then that's going to create a CI/CD lab folder. So, we want to go into that. And then from here, I would
5:22
suggest that you remove the current git repository and create your own. So, we'll do rm-rf.get.
5:31
And then we'll do git init-b main, which will create a new repo on
5:37
the main branch. And then let's open up VS Code or whatever uh you know whatever
5:42
editor you want to use. All right. So let's take a quick look at the code. If we look at the package.json,
5:48
we have a few dependencies. We have express which is our, you know, regular uh web framework. And then we have our
5:56
dev dependencies including TypeScript. TSX will will allow us to run TS files
6:02
in our node environment. And then we have a couple custom, you know, types for Node and Express. And then also I
6:09
added this engines node version 24 which is the current long-term support version. That's what I have on my
6:16
machine. That's also what I want render to run. That's also what we'll configure our GitHub um uh virtual machine to run
6:23
as well. So there's a few scripts as you can see. There's a dev script to to so
6:29
that tsx will watch the serverts file. And if you look in our source folder
6:34
there's just a serverts and an app ts. That's our entire uh source code. But let's run the server. So I'm going to
6:40
open up a terminal within VS Code. And before we run it, we do have to install our dependencies. And to do that, you
6:47
could do npm install, but I'm actually going to do npmci. And the basically the difference here is
6:53
when you run npm install, it can change your package lock json file and the
6:58
versions. CI is going to stay true to the versions that are in that lock file. And that's what you want to run when you
7:05
set up like your build command and render and so on. Wow, that that wind is strong. I don't know if you guys can hear that. So now let's run npm rundev.
7:15
And that's going to open on localhost 3000. Feel like my house is going to blow away. And the homepage is just
7:22
this. I mean it's such a a simple it's not it's just a boilerplate, right? Because it the code doesn't matter. This
7:28
could be a giant code base. I I would be showing you the same thing. Um, but the more code the more confusing it would be
7:34
to you guys. So yeah, if we look at the appts file, we're basically just initializing express and then I have two
7:42
routes. One is the home route which just renders some HTML as we just saw and
7:48
then there's just an API health route that just gives a a healthy response or 200 response. So that's it. That's our
7:55
entire project here. Now there is a test folder with an app.est test file and
8:02
most of this is just set up because it is an express server. So we have a before function that runs before the
8:08
tests and sets up our server. So app.listen zero just means the next available port and it's in a promise
8:15
because running the server is asynchronous and it could take a couple milliseconds. So if it if there's an
8:21
error, it will reject from the promise and it will resolve if not. And then we just get the the address. we get the
8:28
base URL and then after the test run that server closes. Now the only actual
8:34
test is this one here which is just to make sure that the API/halth endpoint returns a healthy response. So
8:41
it just looks to make sure that it's a 200 response. Okay, so that's our test.
8:46
Now if we go back to package.json, these three scripts right here, these
Our Verify Script
8:53
commands are very important because these are basically our checks. We have type check which will look for any
8:59
TypeScript errors. We have test which will use the node testr runner to look at that app test file and any tests that
9:06
are in it. And then we have our build command which will run the TypeScript compiler because it needs to compile the
9:13
TypeScript to JavaScript. Right now those are that's what we actually want to do from within our GitHub actions.
9:20
But typically we don't want to just have these all three separate scripts or commands. We want to create one verify
9:27
script and then run all three of those. So let's create a script called verify.
9:32
And then we're going to do npm run type check double amp or samp. And then we're
9:38
going to do npm test. And then we're going to do npm run build. So it's going
9:43
to do all three of those things. And we'll save that. And then we can try that out. So you can stop the server and
9:51
we'll clear this up. And then we'll do npm run verify.
9:56
And it should go through cleanly. You can see that the the one test we have passed. There's no Typescript errors.
10:03
And if you look up in the in the explorer here, there is a disc folder with our two JavaScript files. Okay. So,
10:10
our source code was TypeScript. We ran the build process and we got back our JavaScript with no errors. All right. So
10:17
that's pretty much it as far as what you need to know about this this project. And if we look in our git ignore where
10:24
you know obviously node modules we don't need to push the disc folder any environment variables things like that.
10:30
And then we also have our TypeScript configuration which is very uh very simple very common. And that's it. So
10:37
what I want to do now is push this to GitHub. Right. So we're going to push it. We're going to deploy to render just
10:42
normally. And then we'll create our GitHub actions. So, we're going to go ahead and deploy our project to GitHub.
Push To GitHub
10:49
So, let's create create a new repository here. And I'm just going to call this CIC CD-l.
10:57
All right. Now, we need to to push to it. So, I'm going to grab this git
11:02
remote right here. Okay. Not the whole thing, just this. I I mean, if you want to use the commands, you can. I'm just
11:08
using the VS Code UI. So, let's jump into that. And let's see. First off, we
11:14
got to stage everything. So, I'm going to click on the the get icon here or the version control icon. And where it says
11:21
changes, I'm going to click the plus at the top. That stages everything. And then we'll make a commit. So, let's say
11:27
chore and setup app baseline commit.
11:34
And now I want to add the remote. So to do that, let's do command shiftp and
11:39
just click just type remote. And you should see this get add remote. I'm going to paste in that link and then
11:46
enter and then origin for the name and then publish branch and that should get
11:53
pushed to main. We'll refresh and there it is. Okay. Now to deploy to render is
Deploy To Render
12:00
very simple now that we have that on GitHub. So I'm just going to sign in here with GitHub
12:05
and then I want to create a new web service. So I'm going to go up here to the top right. So you have op bunch of
12:12
options static site web service you can you uh if you want to create a postgres database cron jobs uh workflow
12:21
background worker but we want web service that's what this is and then we want the repository we just pushed which
12:28
is cicd lab and then there's just very minimal configuration that we need to do here we
12:35
come down to build command npm install is the default I actually want to use npm MCI for reasons that I explained
12:43
earlier. And then let's do and npm run build because we obviously we want to
12:49
compile our TypeScript into JavaScript. And then for the start command, we can
12:55
do npm start because we do have that in our package.json. That's going to run the server.js file, the production code.
13:02
And then you can choose the $0 per month here. And then one thing I want to do
13:07
under advanced. So you have an option to put a health endpoint which we do have.
13:13
So slash APIhealth. So it'll monitor that. And then this
13:19
autodeploy it's set to on commit by default which is which is continuous
13:24
deployment. Right? If you want continuous delivery and you want a human to be to you know inspect everything
13:30
before the merge and so on or before the deployment you can set that to off. Now
13:35
once we add our checks right we set up our GitHub actions. We set up our verify. We're going to change this to
13:42
after CI checks pass. That way, it only deploys if if the CI passes. But for
13:48
now, we're just going to choose on commit. And we'll deploy the web service. And this should be pretty quick
13:54
because it's a very very small project. All right. So, that took maybe 25
13:59
seconds or so. And you can see it requested node version 24. That's because I had remember I had the engines
14:06
value in the the package.json. JSON and then we get build succeeded. So no
14:12
issues. NPM start to start it up. And then we have a custom domain here. So if
14:17
I click on that, it opens it up and we can see the the homepage, right? And if
14:23
you go to deploys, that's where you're going to see all your deployments from now on. And again, continuous delivery.
14:29
The CD part of CI/CD is is all set up because if you push to GitHub, the
14:35
you're automatically going to see it deploy right here, which is nice. So, what we want to do next is set up GitHub
14:41
actions so that when we when we make a pull request or we push to main that it'll run our verify, it'll basically
14:48
set up a a virtual machine and then go through that process and make sure everything's all right. So now we want
Github Actions Workflow
14:54
to set up GitHub actions which lets us automate tasks in our repo. And ultimately what we want to do is run
15:00
that verify script whenever a pull request is made to merge into main or if it's pushed directly to main and we're
15:07
going to set that up with a workflow file and then we're going to make it mandatory so that those verify checks
15:14
pass on or have to pass both on the GitHub layer and on the render layer. So
15:20
I want to create a new branch for this and that's what we're going to do. I know the first push to GitHub we just
15:26
push directly to main but anything from now on will be through you know we'll make a pull request and then we want to
15:32
create this workflow file and I'll explain all this but what I want you to do now if you're following along is just
15:38
copy all this right here and then let's go into back into VS Code and from here
15:45
I'm going to create a new branch. So down here where it says main, I'm going to click that and let's click create new
15:51
branch and we'll call this feature slash and GitHub github-actions-c.
16:00
Okay, so now we're on that new branch. And then I want to create a new folder
16:06
and this is going to be called.github. Okay, so that's the convention. Don't forget the dot. And then we'll create a
16:12
new folder in there called workflows. And then we're going to create a file in
16:17
there called ci.yiml which is a YAML file. And that's what this this format is. So we'll save this.
16:25
And now I'll just explain what this file does and what this means. So first off
16:30
we have a name just called CI. That's what GitHub will see. And then we have this on block where this is when uh
16:38
basically the job that we have down here the verify job when this will run. And in our case, we're saying if we push to
16:46
the main branch, right? So if we push and branches just main, if you wanted to
16:51
have other branches here, you could do that. But we we're just if we push to the main branch, then also if we make a
16:56
pull request to the main branch as well. So in in those two situations is when we
17:02
want this to run. Permissions, we just have contents read because we just need to be able to read the files and um and
17:10
just run the check. So there's there's nothing else. We just need a read permission. And then here we have the
17:15
jobs block with one job called verify. So it's named verify. And this is where
17:21
it's it it creates that virtual machine, right? And and we're just saying we want to use Abuntu latest for that latest
17:28
version of Abuntu. And we're setting the timeout minutes to 10 so that it does doesn't run indefinitely. Um and then we
17:35
have the steps, right? So it's one job, but there's a couple steps. And the
17:40
first step is to just check out the repository, right? Just check out the code. And then the second is to set up
17:46
Node.js. And remember we specified um version 24 in our package.json. So we
17:53
want this to match that. So Node version 24. And then it'll install the dependencies. So npmci it's going to
18:00
run. And then our verify. So npm run verify. So those are the steps. Now,
18:06
you'll notice that some of these steps have a uses and some have a run. So, basically uses is if you're going to use
18:13
a predefined action, a GitHub action such as checkout, right? So, this uses the
18:19
checkout action. And then this one uses setup node. And then the run is just a a
18:26
command, right? NPMCI or npm runverify. So, that's it. It's a very very simple
18:32
file to um to to understand. So now what we want to do is bring this into our
18:38
repo. And on this particular pull request that we're going to make, it's
18:43
actually going to run. We're going to be able to see this run in actions. We want to go to our our version control here. I
18:50
want to add the CI YML file to staging. So I'll click uh the plus and then the
18:57
commit message. Let's say this is going to be feature colon and we'll say setup um setup
19:05
GitHub actionsci and then I'm going to commit
19:11
and then publish branch and then what that's going to do if I go over back to the the repo this right here and you'll
Our First Checks Run
19:19
see this this box pop up with the compare and pull request and if I click on that
19:25
say create poll request test and then we should see the checks
19:31
actually start. All right, so it actually it already ran. So you can see the green check mark. But we can go we
19:38
can click on this and we can see exactly what happened. First off, it just set up the job, right? And then we have
19:45
checkout repository, which was our first step that we listed in the actions file.
19:50
In fact, if we look right here, so check out repository, that was the first step.
19:55
So that's what ran. Then we have set up Node.js. Okay, that was our second step.
20:01
And you can see it it used version 24. And then it had install dependencies.
20:06
That was our third step we defined which was npmci. And then we have verify application
20:12
which ran npm runverify. And you can actually see the result here. You can see that one test passed. So yeah, it
20:19
does the exact same thing as if we if we were to run npm run verify on our machine, but it does it in the cloud and
20:27
then it just has this post setup node and so on. So it ran the checks, right? And this is in the actions tab. So you
20:33
can always come to this actions tab and let's go back to the pull request because we didn't uh we didn't merge it
20:40
yet. But the checks passed now from where we are right now. Even if
20:45
something like if a test failed or something like that, we could still merge. Uh what I want to do, I'm just
20:51
going to merge this real quick. But what I want to do is make it so that if something failed, then I don't want it I
20:58
don't want to be able to merge. And I want to set that up on the GitHub layer and on the render layer. But let's go
21:04
ahead and delete this branch now. Okay. So now that's in main in remote.
21:10
Now, let's go back to our local and we're going to want to let's close that
21:16
up. We're going to want to go back to main here. So, click on main. And then
21:21
we want to pull. So, right uh let's go back to the version control and pull.
21:27
And now we should have that GitHub folder, the GitHub in main. And now we can delete that sub branch. So, if we do
21:33
command shiftp and if you just type in git, you should see delete branch. And
21:39
then you should see that CI branch that we created. Just click on that and it'll delete it. All right. So now we're all
21:45
up to date. And now we're ready to basically make those checks mandatory.
Github & Render Enforced Guardrails
21:50
Okay. So now we want to make the the verify job that we just created required on two levels. One on GitHub and which
21:57
we we do something called a a rule set or we create a rule set and then we also want to do it on the deployment layer
22:04
with render and that's just a simple selection item. So, let's jump over to
22:09
GitHub and we want to go to our repo. We want to go to settings.
22:15
And then from here, we're going to go to rule sets and we want to create a new rule set, a
22:21
new branch rule set. And let's call this protect main. We want to set it to
22:28
active. Um, we don't need the bypass list. Target branches. We want to target
22:33
the uh include default branch which in our case is going to be main. And then we want we want to enable enable a
22:41
couple things here. One is going to be require a pull request before merging.
22:46
So we're going to check that. We can keep that at zero. And then
22:52
we also want to enable let's see require status checks to pass.
22:59
And then here where it says plus add checks, just type in verify. And then
23:06
you should see this right here from your GitHub actions. Verify. We're going to check that.
23:14
And then I think I think that's all I all we need to do here. Yeah. So let's
23:19
click create. And now we have that rule set where it won't let us merge into
23:25
main if that doesn't pass. So that takes care of GitHub. Now for for a render for
23:31
that layer, we're going to jump back over to our dashboard and we're going to go to So I'm already in my CI/CD lab.
23:39
I'm going to click on settings and then we want to go to where it says autodeploy.
23:46
So right here and I remember I showed you this when we did our deploy initial deployment. It's set to on commit, but
23:52
I'm going to edit it and I'm going to set it to after CI checks pass. So
23:58
that's just what it sounds like. The CI CI checks have to pass in order for it to deploy. So let's save changes there.
24:05
And now we basically have two levels of protection. One for get merging into the
24:10
main branch on GitHub and one for actually deploying. So now what we want to do is test the guardrails that we
Github Gate Test
24:16
just created. So if you go to the gist, you're going to see this demonstrate a failing test. I want to go ahead and
24:22
grab this test and then we're going to go to VS Code and we're going to create uh a new branch. Click on main create
24:29
new branch. Let's call this feature slash and we'll call this version dash
24:34
endpoint for the the branch name. And then I'm going to go to the test file, go to the very bottom and paste in this
24:41
test. Now what this does is it just checks the slappi/version
24:46
endpoint to see if it returns a the current version which will fail right now because we don't have that endpoint.
24:53
In fact, we can check that. So let me just bring this up and we'll say npm run verify and you can see that
25:01
we get a failing test. Now what I want to do is is make a pull request and try
25:07
to merge this into main with this failing test to make sure that our you know our our rule set our guardrails
25:14
actually protect us. So let's uh let's go back to the version control here. Let's click plus to stage the file and
25:21
then make a commit. I'll call this test and say test endpoint
25:28
um endpoint failure commit and then publish branch and then I'll go back to uh the repo
25:35
here. Okay, we're not getting a box. So what we'll do is click on the branch
25:40
dropdown and go to feature version endpoint and then we're going to go to contribute
25:46
and then open pull request. Okay. So from here I'm going to click create pull request.
25:53
And now we should see that check fail. When it's yellow that means it's it's
25:59
running or it's cued. So now it's running and I can actually click on it and we can watch it and it should fail
26:06
right here. Yep. See? So now it has a red X under verify application.
26:14
And now if I go and that's under actions. Now, if I go back to my poll request,
26:20
it's it's failed. It's a red X and I can't merge it. Okay, that's because of that rule set that I that I set. So,
Render Gate Test
26:28
there's no way that I can merge this. Now, it didn't even get the chance to get to render because it didn't it
26:34
didn't merge, right? So, what I can do is shut off the that that rule. So, I'll
26:39
go back to settings and go to rule sets and go to protect main. And where it
26:48
says active, I'm going to choose disabled and click save. Now that I disabled that, if I go back to my pull
26:54
request, it's still failing, right? But I do have this merge pull request button that I
27:01
can now click. So, I'm going to click it and confirm.
27:06
Now, usually when you when you do this, when you merge to main, if you go to
27:12
render and you go to did I close it? Where is it? And you go to deploys, right? It
27:20
should start. So, I'm going to refresh. And you can see that there's there's no
27:25
deploy starting. And the reason for that is because of that setting that I did uh
27:30
a few minutes ago, which is this right here after CI checks pass, right? So the
27:37
CH the CI check didn't pass. That that red X, right, that we saw, it didn't
27:43
pass. So the deploy keep losing my spot. The deploy didn't
27:48
even start. Okay, so that proves that the the render guardrail is also
27:53
working. Now, there's a couple things I could do here. I'm actually going to revert. So,
27:58
I'm going to click revert here. Request to revert.
28:05
And this is just going to basically bring it back to to where it was. All right. So, the checks pass
28:12
because I reverted it. Okay. So, I'm going to merge. Uh, yeah, I'll just delete this branch.
28:20
So, we should be clean. Now, if I go to code and I go to test, we should not
28:25
have that version test anymore, which we don't. All right, cool. And yeah, I'll
28:32
just keep that other the version endpoint because now what I'm going to do and I have to turn the checks back on
28:37
or the rule set back on. Yeah. So, we want this enabled.
28:47
Okay. Okay. And then what I'll do is go back to VS Code
Passing The Gate
28:53
and let's see. We're still on version endpoint. We're still on that branch and we have the version test here. Now I'm
29:00
going to add the actual route so that the test doesn't fail. So I'm going to go back to uh appts
29:07
and now in the gist I have the actual route. So we can just grab it here.
29:15
And then let's paste that right here. Okay. And now if I run verify, it should
29:22
pass. Let me clear this up and do npm run verify.
29:28
Okay. So it passes. We got two passing tests. Now if I go ahead and let's see
29:35
let's add stage that feature added version
29:41
endpoint commit sync changes.
29:47
Okay. So now if I go back to the repo here and we go to our this branch here
29:54
the version endpoint we should have we look in app TS. Yep. So, we have that
30:02
endpoint. Now, let's make a PR. So, I'm going to go to contribute, open pull request, create pull request,
30:11
and now this should pass, right? I have the guardrails, but that we have everything's passing because we have the
30:17
test, we have the endpoint, and everything should be good. And then it should also deploy to render.
30:23
So, that passes. Let's merge, confirm,
30:30
and then we'll delete branch. Okay. So now since that passed and we have no
30:36
errors or anything, if we go back to render, we should see under deploy, we should see that start any second.
30:46
And there it is. So now it's starting up again. And if that if that had failed,
30:52
even if we had merged into Maine, this wouldn't have started up. So, we should be good. Now, let's go
30:58
back to VS Code. Let's switch to main
31:03
and let's pull from main.
31:08
And there we go. So, now we're up to date. Let's delete delete that other branch.
31:15
So, hopefully you can see, you know, what's going on here. Here you can see the the guardrail we set up the verify
31:22
script and how it h it has to pass right if we have any failing tests it we're
31:27
not we're not going to be able to merge we're not even going to be able to get to the point to deploy so the last thing I want to show you is preview
Preview Environments
31:33
environments these will let you try out a change in the browser before it goes live and when you open a pull request
31:39
render will give you a temporary deployment with its own URL so you can test it you can share it for feedback or
31:46
whatever you want and production stays unchanged and then once you merge or close the PR render cleans it up
31:52
automatically. So just making the PR, leaving it there, not merging it. You'll have a URL that you can that you can
31:59
check everything out on. And it's it's with the free account, so you can use it, you can test it out. And what you'll
32:07
do is go to your project and render. Go to previews and then edit and then automatic. Or you can do manual where
32:14
you select uh certain PRs. But I'm going to do automatic. And then that way every PR I create will
32:21
have its own URL. Now, the production domain, this is what it looks like. And what I'll do is make a change in VS
32:29
Code. So, I'm going to go and create a new branch. And I'll just call this uh
32:34
feature slashhello. And then I'm going to make a change by
32:39
going to app.pt.ts and just going under the H1. And let's create an H2. And I'll
32:45
just say hello world. Yeah, we'll just do that. It doesn't
32:51
really matter the change. And then we'll go ahead and stage that. Make a commit.
32:58
Let's say feature uh hello world and commit publish
33:06
branch. Okay. Let's go back to the the GitHub repo.
33:14
And we'll go to our feature hello and then contribute. make
33:19
a poll request. We're going to create the poll request, but we're not going to merge just yet.
33:26
All right. So, right here it says Brad Traversy deployed to feature/hello. Now, this is actually a link. I'm going to
33:32
open it in a new tab. And now you can see we get our hello world H2. And it's
33:37
a separate domain. It actually has the PR in it. And if I go back to the the the production domain here and refresh,
33:45
there's no Hello World. So it's a completely different environment and I can I can share this if I want. If I
33:50
have a team and I want to show them something, I can easily share it. And then once I merge it, right? So I'm
33:56
going to merge the hello world into main. All the checks passed and everything.
34:03
And then I'll delete that branch. Okay. Now, if I go back to that custom
34:09
domain and refresh, it's gone. It doesn't work anymore, which is what I want. It's just a preview. And if I go
34:14
to the to the production domain now and refresh, well, it didn't deploy yet. It should uh
34:21
it should start the deployment in render. So if we go up to deploys, there
34:26
it is. So it started up. And once that's done, then the you know the public domain or the production domain will
34:32
have hello world. All right. So hopefully this helped you guys out. But I know that, you know, a lot of people
34:38
talk about CI and and how important it is, but I feel like nobody really explains it in a simple way and and
34:44
shows you simple examples. So that's what I wanted to do. Hopefully I succeeded in that and and you learned a
34:50
little bit. And this this is stuff is really important in the age of AI because a lot of people most people
34:57
aren't writing most of the code that that they push. So you can use do this
35:02
along with like code code code reviewers and have a system that that really works that gives you protection. But that's it
35:09
guys. Hopefully you enjoyed this and I will see you next

