---
layout: post
title: Creating a Self Updating Portfolio
date: '2017-10-9 09:00:00 +0530'
categories: web
comments: true

---
As a developer, the best way to showcase his/her talent is via a portfolio website. Many of us want to build a "perfect" web portfolio which looks aesthetically pleasing along with showcasing our skills and work effectively.
But once we build our web portfolio we often forget to keep it up to date. Most of us even have CVs that is not up to date for many years. This happens because in our busy lives we don't want to spend that additional time keeping our CVs/portfolios updated.

<br>
<div style="text-align:center">
  <img src="/blog-atul/assets/self_updating_portfolio/cv_start_meme.jpg" style="width: 60%;display: inline;">
</div>
<br>

In the world of bots and automation, I thought why cant I come up with a website for myself that automagically updates on its own for free.
The end goal is a magical website that somehow keeps itself updated with the latest projects that we do and it should do so without the owners manual intervention.

Before we dive in to the automation part lets first get some basics right.

## Github as a host
First thing that comes in our mind when we think of building a portfolio website is that where should we host it?
Enter Github; Most of you use Github for hosting your open source projects. Apart from providing free hosting for projects.
Github also provides a free service called Github Pages. Github pages can be used to host your portfolio website or even individual webpages for your projects.
To know how to setup github pages, visit: https://pages.github.com/

So Github now becomes a platform to host our projects and our portfolio website.

## Building the site
Next step is to actually build our portfolio website.
A basic portfolio website may look like this.
 <br>
 <div style="text-align:center">
   <img src="/blog-atul/assets/self_updating_portfolio/portfolio_wireframe.jpg" style="width: 100%;display: inline;">
 </div>
 <br>
 You can use a static site generator like Jekyll or even a predefined boilerplate to start building your website. Or if you want to build a completely custom website like me, you can do so with HTML and CSS.

## Data points in a portfolio
If you take a look at the structure of the website, you could see the following data points:

- Profile picture
- Summary of the person
- Projects and their details
- Blog feed
- Social media feed

#### Profile picture
It is recommended that you upload your profile picture at one place and use the same everywhere. This way you update at one place and it reflects everywhere.

There are few options:

- **Gravatar** : gravatar.com is a website which hosts your profile picture and provides you a link. Gravatar is used by many blog sites to fetch your profile image based on your email id.
- **Github** : Just upload your profile picture on github and use the same link on your blog also.

#### Summary of the person
The summary of the person usually doesn't change from time to time. Hence this part can remain static and just store the data in some JSON or as a static content on the website.

#### Projects and their details

I assume we are hosting all our projects source code on Github. Now, Github provides all the projects you have hosted on it via Github APIs. To have access to Github APIs one should first subscribe to the Github developer program.
In general, we would want the following details of a project via the API:

- Project name
- Project description
- Link to the project
- Stars and forks
- tags related to the project

Although you can use the REST API provided by Github to get all these details for all the projects, but that would require many calls to the Github API server.
Instead Github launched its V4 GraphQL version of the API via which you can just make a single API call and get all the details of all the projects.

A detailed reference to the GraphQL Github API can be found here https://developer.github.com/v4/guides/

The query that I use is as follows:

{% highlight html %}

{
      user(login: "master-atul") {
        bio,
        avatarUrl,
        repositories (first :100 privacy: PUBLIC) {
          nodes {
          	name,
            url,
            stargazers{
              totalCount
            },
            owner {
              login
            }
            forks {
              totalCount
            },
            homepageUrl,
            description,
            repositoryTopics (first: 100){
              nodes {
                topic {
                  name
                }
              }
            }
          },
          pageInfo {
          	hasNextPage,
            endCursor
        	}
        }
      }
    }
{% endhighlight %}



You can test this in the GraphQL explorer provided by Github.
This helps us get all the data we need with a single API hit. 🤘🏻

### Getting around Rate Limits on Github API

But unlike me, many of you might have lots of users and lots of users mean lots of API hits. Hence, you might reach your API rate limit and new users might not be able to see anything.

To get around API Rate limits, we will use travis.
Travis CI provides free cloud based CI for open source projects.

Hence the idea is:

- Travis CI hits the Github API server every day once or twice and gets the JSON with our project details.
- Travis CI now pushes the generated JSON onto our website repo
- Travis will then rebuild the website and redeploy.


## Scripts

### Github API script
The Github scripts purpose would be to get all the latest projects info from your github repo and put it into a json file that we can use in the website.
I am going to use node to write the github api script, primarily because its easy to do fetching and data transformation in node.

My script looks something like this:

**`projects_updater.js`**

~~~js

var rp = require('request-promise');
var result = require('lodash/result');
var each = require('lodash/each');
var fs = require('fs');
var path = require('path');

var args = process.argv.slice(2);

const GITHUB_API_V4_READ_TOKEN = args[0];

const _githubFetcher = (query) => {
  const requestBody = {query};
  return rp({
    uri: 'https://api.github.com/graphql',
    method: 'post',
    json: true,
    headers: {
      'Authorization': `bearer ${GITHUB_API_V4_READ_TOKEN}`,
      'User-Agent': 'Request-Promise'
    },
    body: requestBody
  });
};

const _generateProjectReposQuery = (totalResults = 100, nextId = null, repositoryType) => {
  const offset = nextId ? `after:"${nextId}"` : '';
  return `{
      user(login: "master-atul") {
        bio,
        avatarUrl,
        ${repositoryType} (first :${totalResults} ${offset} privacy: PUBLIC) {
          nodes {
          	name,
            url,
            stargazers{
              totalCount
            },
            owner {
              login
            }
            forks {
              totalCount
            },
            homepageUrl,
            description,
            repositoryTopics (first: ${totalResults}){
              nodes {
                topic {
                  name
                }
              }
            }
          },
          pageInfo {
          	hasNextPage,
            endCursor
        	}
        }
      }
    }`;
};

const _recursiveProjectGetter = (queryParam, queryResult, repositoryType) => {
  const query = _generateProjectReposQuery(queryParam.totalResults, queryParam.nextId, repositoryType);
  return _githubFetcher(query).then((response) => {
    const nextPageInfo = result(response,  `data.user.${repositoryType}.pageInfo`, {});
    if (response.errors) {
      throw response.errors;
    }
    if (nextPageInfo.hasNextPage) {
      queryParam.nextId = nextPageInfo.endCursor;
      queryResult.push(response);
      return _recursiveProjectGetter(queryParam, queryResult, repositoryType);
    }
    queryResult.push(response);
    return queryResult;
  });
};

const _getMergedRepositoryInfo = (resultArray = [], repositoryType) => {
  const nodes = [];
  each(resultArray, (res) => {
    const resultNodes = result(res, `data.user.${repositoryType}.nodes`, []);
    nodes.push(...resultNodes);
  });
  return nodes;
};

const _writeJSON = (filename, data) => {
  const filePath = path.resolve(__dirname, '../app/assets/json', filename);
  fs.writeFileSync(filePath, data);
  return filePath;
};

const _getAllProjects = (repositoryType) => {
  const queryResult = [];
  const queryParam = {
    totalResults: 100,
    nextId: null
  };
  return _recursiveProjectGetter(queryParam, queryResult, repositoryType).
    then((data) => _getMergedRepositoryInfo(data, repositoryType));
};

// MAIN STATEMENT THAT EXECUTES THE ABOVE FUNCTIONS
// BASICALLY THIS GETS ALL THE REPOS AND CONTRIBUTIONS FROM GITHUB

Promise.all(
  [
    _getAllProjects('repositories').then((data) => _writeJSON('projects.json', JSON.stringify(data))),
    _getAllProjects('contributedRepositories').then((data) => _writeJSON('contributions.json', JSON.stringify(data)))
  ]
).then((filepaths) => console.log('updated files', filepaths)).catch((err) => {
  console.log(err);
  process.exit(-1);
});

~~~

To use this script, just run
`node scripts/projects_updater.js <GITHUB_API_V4_READ_TOKEN>`


#### Getting the GITHUB_API_V4_READ_TOKEN

0. Login to http://github.com
1. Open up https://github.com/settings/tokens
2. Click on Generate new token.
3. Give these permissions: public_repo, read:user, user:email.
4. Make sure you copy the token and keep it safe.


### Travis CI - Deployment script

I was using webpack to build the final static asset for my website.
I am using travis to automate the build whenever we push to the repo.
For this I wrote a build script that needs to execute on every push to repo.

**`build.sh`**

~~~sh
#!/bin/bash
set -e # Exit with nonzero exit code if anything fails

SOURCE_BRANCH="source"
TARGET_BRANCH="master"

function updateProjects {
  yarn install &&
  node scripts/projects_updater.js ${GITHUB_API_V4_READ_TOKEN} &&
  yarn build
}

function create_all_branches {
    # Keep track of where Travis put us.
    # We are on a detached head, and we need to be able to go back to it.
    local build_head=$(git rev-parse HEAD)

    # Fetch all the remote branches. Travis clones with `--depth`, which
    # implies `--single-branch`, so we need to overwrite remote.origin.fetch to
    # do that.
    git config --replace-all remote.origin.fetch +refs/heads/*:refs/remotes/origin/*
    git fetch
    # optionally, we can also fetch the tags
    git fetch --tags

    # create the tacking branches
    for branch in $(git branch -r|grep -v HEAD) ; do
        git checkout -qf ${branch#origin/}
    done

    # finally, go back to where we were at the beginning
    git checkout ${build_head}
}


# Variables
REPO="https://github.com/master-atul/master-atul.github.io.git"
SSH_REPO=${REPO/https:\/\/github.com\//git@github.com:}

git config user.name "Atul R"
git config user.email "atulanand94@gmail.com"

#this is necessary because by default travis only shallow clones the master branch
create_all_branches &&

git checkout $SOURCE_BRANCH

#updating the project by running the build
updateProjects

#copying important assets to a temporary directory
cp -rf bundle ../portfolio_dist
cp CNAME /tmp/CNAME
cp README /tmp/README

#cleanup before changing the branch
git stash
git clean -fd

git checkout $TARGET_BRANCH
cd ..
echo "removing old dist"
rm -rf ./master-atul.github.io/**
cd master-atul.github.io
echo "copying new dist"
cp -rf ../portfolio_dist/* .
echo "current directory ${pwd}"
ls ../portfolio_dist
cp /tmp/CNAME CNAME
cp /tmp/README README

git add -A .
set +e
git commit -m "Commit new bundle to ${TARGET_BRANCH}"
set -e
git push $SSH_REPO $TARGET_BRANCH

~~~

To run the script:
`bash ./scripts/build.sh`

In github we need to keep the final dist on the `master` branch of the portfolio repo.
So the TARGET_BRANCH is the master branch and we keep our uncompiled source code in the SOURCE_BRANCH.
My SOURCE_BRANCH is `source`.

Hence, the flow is we keep our source code in SOURCE_BRANCH and keep pushing any changes that we make there. The travis CI will pick up the source code from SOURCE_BRANCH and then build the static assets and push the final built assets onto TARGET_BRANCH. Github will pick up the code from master branch and will serve our website.
<br>
<div style="text-align:center">
  <img src="/blog-atul/assets/self_updating_portfolio/travis_portfolio.png" style="width: 100%;display: inline;">
</div>
<br>


Now I just need to sit back and focus on my work and my portfolio will take care of updating itself 🍷 😎

Hope this helps people create awesome automated portfolios ! Cheers 🍻 🌮

**You can see the live example of this at** http://atulr.com


<br>
<div style="text-align:center">
  <img src="/blog-atul/assets/self_updating_portfolio/resume_final_meme.jpg" style="width: 60%;display: inline;">
</div>
<br>

<br />
<br />
<hr />
<br />
{% if page.comments %}
<div id="disqus_thread"></div>
<script>

/**
*  RECOMMENDED CONFIGURATION VARIABLES: EDIT AND UNCOMMENT THE SECTION BELOW TO INSERT DYNAMIC VALUES FROM YOUR PLATFORM OR CMS.
*  LEARN WHY DEFINING THESE VARIABLES IS IMPORTANT: https://disqus.com/admin/universalcode/#configuration-variables*/
/*
var disqus_config = function () {
this.page.url=http://www.atulr.com/blog-atul/web/2017/10/9/self-updating-portfolio.html;  // Replace PAGE_URL with your page's canonical URL variable
this.page.identifier = SELF_UPDATING_PORTFOLIO; // Replace PAGE_IDENTIFIER with your page's unique identifier variable
};
*/
(function() { // DON'T EDIT BELOW THIS LINE
var d = document, s = d.createElement('script');
s.src = '//atulr.disqus.com/embed.js';
s.setAttribute('data-timestamp', +new Date());
(d.head || d.body).appendChild(s);
})();
</script>
<noscript>Please enable JavaScript to view the <a href="https://disqus.com/?ref_noscript">comments powered by Disqus.</a></noscript>

{% endif %}
