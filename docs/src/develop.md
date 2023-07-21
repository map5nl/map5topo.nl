# Development

Development takes place using the `map5topo` GitHub repo. For now this is a single 
repository that contains all that is needed to prepare the data, generate the map (tiles) and use
various tools to inspect and apps to view (map viewers). 

There are three main servers in the development chain:

* topo.map5.nl the DEV server
* test.map5.nl the TEST server
* map5.nl the PROD server

The latter two are identical, only the final tiling (in GeoPackages) 
is done on TEST and then copied to PROD.

## 0. Startup (once)
 
* the repo is currently private, you need to be added with proper rights
* `git clone https://github.com/map5nl/map5topo.git`

## 1. Keeping current

Before changing any file be sure to do a `git pull` to become current with the
latest version of the repo. 

Use  `./refresh-git.sh` if you want to wipe any local changes. 
**Warning: your changes are lost forever!**.  
Instead you may do `git stash` to keep your changes locally or use branching.

## 2. Issues and Milestones

GitHub Issues and Milestones are used track progress and releases. 

### Working with Issues 

Issues can be opened by anyone. They usually represent a new feature or a bug. 
Through Labels, Issues can be tagged with a Priority and other characteristics like Bug, Enhancement, Documentation.

When changing and commit/pushing code to the repo, you are strongly encouraged to add the Issue number 
in the Commit message. That way we know that an issue is worked on and the changes related to the issue.
The issue number is prefixed with a 'hash' symbol. For example:

```
git add tools/mapnik/styles-map5/mystyle.xml
git commit -m "#128 changing zoom levels"
git push

```
 
For trivial changes or quickfixes there may not be an issue number, so not required.

### Working with Milestones
A Milestone is basically a list of Issues with a name, state and description. 
In the GitHub UI you can add or remove an Issue to/from a Milestone.
A Milestone corresponds to a Release. Milestones, thus Releases, are named YYYY-MM, for example 2023-05. 
See our Milestones here: https://github.com/map5nl/map5topo/milestones.

This way we know "what went into a Release" which greatly helps making release notes. 
If an Issue is not finished, it can be moved to a next
Milestone, thus Release.

## 3. GitOps Automation

The "DEV" server is topo.map5.nl. This server always has the latest version of the GitHub repo `main` branch.
Whenever something is changed in the repo, that code is refreshed (via a GitHub Workflow). Further Workflow actions depend on 
the directory the change was made. 

Whenever something is committed/pushed, a GitHub Workflow is started (from within GitHub) that applies
to the directory where a change was made. If multiple directories are changed, multiple Workflows are started.

In general for `services`, the (`docker-compose`) service will be restarted. If a new version is configured, the Docker
Image is downloaded first. 

For changes under `tools` the following applies:

* `tools/mapnik`: the MapProxy service will automatically reload the Mapnik config (Layers/Styles)
* `tools/etl`: no effect, any ETL needs to be run manually on the server, as these may be long-duration processes
* `tools/dem`: no effect, as these may be long-duration processes
* `tools/qgis`: no effect, basically the QGIS Inspector project connects to the `service/featureserv` via OGC Feature REST API

## 4. Inspection

Several tools and apps are available to inspect data and maps.

* https://topo.map5.nl/app/ several apps 


Style/map inspection:

* the main inspection app is https://topo.map5.nl/app/sidebyside to compare among versions
* `map5topo - DEV - DIRECT` is a Layer that renders tiles from the DEV server without caching, so the Mapnik Styles are directly applied.
* this way you may compare your changes with TEST and PROD 
* NB:  `map5topo - DEV - DIRECT` renders tiles 256x256 directly, without metatiling and gutter. Reason is performance. This impacts label placement!

Data inspection:

* QGIS: start the project from `tools/qgis/map5topo-inspector.qgz`
* PostGIS: use PGAdminIV webapp to inpect PostGIS data: https://topo.map5.nl/pgadmin


## 5. Documentation

You are encouraged to write comments and READMEs, but the main documentation is under `services/apps/www/doc`. Docs are in Markdown. 
[Docsify](https://docsify.js.org/) is used for direct rendering .md files in the browser, so no build is required. We may later switch to something
like `mkdocs` but this is convenient for now.
