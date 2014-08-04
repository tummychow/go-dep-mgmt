# README

As any gopher knows right now, dependency management in Go is... rather a hot topic. Go has yet to find its Bundler equivalent. I believe in the long run we'll get there. But for now, we have a lot of choices. This list attempts to summarize the differences between the choices we have. I have no real experience with them so far, so it's mostly docs right now. Contributions are welcome, see the bottom.

For a general purpose list of Go related stuff, hit up [awesome-go](https://github.com/avelino/awesome-go).

## [godep](https://github.com/tools/godep)

Notable for being one of the most popular options right now. I know that [Drone](https://github.com/drone/drone/issues/177) and [syncthing](https://github.com/syncthing/syncthing) are using it. Also, a recent digital ocean blog post mentioned that they might be using it for their internal Go projects.

godep's preferred specification format is JSON. It dumps all your vendored packages into a Godeps directory at the root of your package. The general expectation is that the code will be checked in with the rest of your stuff once you start using godep.

- `godep save` to count up the versions of any global packages you're using, and vendor them all plus a JSON file for the exact versions you're on. Then you're supposed to commit the entire Godeps folder in your VCS of choice.
- prefix `go` commands with `godep` to add your vendored packages to the front of the `GOPATH` for build/install/test
- `godep restore` to check out global packages to your currently vendored versions. Good for putting your global `GOPATH` in a consistent state vs your current project.
- `godep update` to move vendored versions forward to the current global versions. Sorta the opposite of restore.
- `godep get` is recursive godep-aware `go get`. It installs packages globally, but it'll invoke godep for any of those packages that use godep.

## [gom](https://github.com/mattn/gom)

## [gpm](https://github.com/pote/gpm)

## [gopm](https://github.com/gpmgo/gopm)

## [goat](https://github.com/mediocregopher/goat)

## [goop](https://github.com/nitrous-io/goop)

## [gopkg.in](https://github.com/niemeyer/gopkg)

## Roll your own

Ah the classic solution of rolling your own vendoring. A pretty viable one right now considering how young the community is. It's not that hard to make your own vendoring script, and at least that way you know exactly how it works. Despite that, I wish more big projects would evaluate the options that exist. Even if they dismiss all of them as unsatisfactory, that still tells us something useful as a community - that all our current options have some work to do. I say "big projects" because they often have the most community attention, and need dependency management more than anybody else.

A few good options for rolling your own:

- shell scripts to pull the dependencies into a local vendoring dir, and a makefile to invoke them on a regular basis. I'm pretty sure this is what Docker does, so that's like the number one public Go project right there, quite the endorsement. Packer also does this last I checked. I imagine hashicorp's other Go projects (serf/consul/terraform) are doing the same thing, but I have not checked.
- [git subtrees](https://github.com/git/git/blob/master/contrib/subtree/git-subtree.txt). This is what Soundcloud advocated in their gophercon presentation (I really enjoyed this presentation by the way, watch it [here](http://www.confreaks.com/videos/3434-gophercon2014-best-practices-for-production-environments)).
- git submodules. Ehh... a lot of the internet considers submodules to be broken by design. The soundcloud presentation also advised *against* it.

All my experience is with git, but if you have analogous suggestions for other VCSes, I'm up for a pull request.

# Good blog posts

Please submit others you can think of! (You're allowed to plug your own blog, assuming the article is actually good and interesting) No links to tweets please; they express community attitude but there's not enough substance there for a good read.

- http://dev.af83.com/2013/09/14/a-journey-in-golang-package-manager.html
- http://areyoufuckingcoding.me/2012/04/16/dpendency-hell/
- http://nathany.com/go-packages/

# Contributing

Any contributions are welcome, notably:

- any high-profile Go projects that are using a given dependency management tool (use your discretion for "high-profile". I usually call it on 1000+ stars, assuming it's on GitHub.). If you can find a discussion between project maintainers about what tool to choose, that's a great bonus, because it tells us what those gophers are thinking about in their decisions.
- strengths and weaknesses of the tools you've used, derived from *actual* experience. We're talking Stack Overflow's "good subjective" here. Unfounded ranting about a tool's particular CLI or design decisions is thoroughly unwelcome. This list is supposed to help gophers as a community!
- Objective summary of tool documentation. This is written from the standpoint of a gopher who's asking: "I'm not new to Go, but now I'm in, or starting, a big project and I need dependency management. If I started using `insert-tool-here` right now, what would I have to do?"
- more dependency managers! List them all!
- good articles and blog posts on Go dependency management, things that are worth reading for someone more deeply interested in the subject

# License

MIT
