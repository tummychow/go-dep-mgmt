# Pros and Cons

High level pros and cons of each dependency tool I mentioned, ignoring matters of CLI and command structure. I'm focusing on the design decisions and the things that are easy/hard for each tool, based on the docs that are available. These are highly subjective and reflect my point of view on them, but I'll try to take a balanced stance.

## godep

###### Pros

- `godep save` makes it easy to drop godep into an existing project
- popular. Like it or not, user count matters, because the ecosystem is young.
- `godep get` provides recursive invocation of godep, which is important if godep "wins" the dependency manager "war" (war in quotes because we're all friends here, right?). So far I think it's the only one that has thought this far ahead. Other dependency managers - if you support this, you should document it! It's a key feature!
- godep never actually pulls packages from the internet, it just copies them from your global GOPATH. So it supports all the same VCSes as the go tool.

###### Cons

- non-declarative. You have to get your project into a good state and then `godep save`. To "get your project into a good state", you would have to manually check out each dependency to the desired version in your global GOPATH. It doesn't seem like godep provides a way to read a standalone `Godeps.json` and save exactly those versions to your local vendored directory.
- `godep restore` vs `godep update` was a bit confusing. I had to read the source code for this. It could use some more docs (users of godep, give them a hand and submit a PR! I end up using godep, I'll do it myself.)
- if you want something in your vendored directory, you have to either copy it from the global GOPATH, or put it there manually. godep doesn't do any of the cloning itself. Your dependencies will always enter your global GOPATH first before godep can save them. (Some might refer to this disparagingly as "pollution".)

## gom

###### Pros

- simple bundler-like CLI. Make a Gomfile, `gom install` and go
- Gomfile supports some interesting edge-case applications, like dependency grouping (a la bundler) and specifying an OS for each dependency
- use `gom gen` to make a Gomfile from your current dependencies

###### Cons

- Gomfile syntax is Ruby-like but not actually Ruby. You could end up using Ruby syntax and getting surprised that it doesn't work. Plus it's extra maintenance effort to provide a custom parser for a nonstandard format, and not all the features of the format are documented.
- `gom install` clobbers `go install`, so you have to `gom exec go install` instead. I know, I know. It's nitpicking.
- no support for svn (but it does support bzr)

## gpm / gvp

###### Pros

- dead simple. Anyone who's ever used a shell should be able to understand the essence of what these tools do.
- because shell script is interpreted, gpm/gvp have a plugin system for adding new commands at runtime. That's very difficult to do if your tool is written in go. And it's not another case of YAGNI; useful plugins actually exist.
- `Godep` format is a simple whitespace-separated table with hashes for comments. Not a standardized format, but so trivial to parse that it might as well be.
- since the only means of cloning/updating is via `go get`, gpm supports all the same VCSes as `go get`.

###### Cons

- written in bash rather than go. But to be fair, shell script is arguably the right language choice for such a simple tool.
- due to the limitations of bash, you have to `source gvp in` rather than just `gvp in`. Understandable, but more characters to type and a bit awkward.
- gvp takes effect on a per-shell basis rather than a per-command basis. The statefulness (tying behavior to a shell environment) might not appeal to you. But I bet you could make a plugin for `gvp exec` that solves this problem.
- Godep format is lacking some of the bells and whistles provided by other tools. But a fancier format (eg YAML) would be hard to parse in bash.

## gopm

###### Pros

- bundler-like CLI, although the commands have different names from what bundler users might expect. Make a .gopmfile, `gopm get` and hack away
- use `gopm gen` to make a .gopmfile from your current dependencies
- `gopm update` can update the gopm binary itself! Cmon, that's some thinking ahead right there.
- integrates with LiteIDE. If you're a user of that IDE, more power to you!

###### Cons

- INI format is unstandardized. .gopmfile also looks a bit dense; not clear if whitespace inside values is supported for readability++
- no bzr support

## goat

###### Pros

- uses YAML. In my opinion, it's the perfect balance between being concise enough for human writing, and being powerful enough for nested data structures. Being standardized is also a big plus.
- same bundler-like CLI as other tools. Make a .go.yaml, `goat deps` and hack away

###### Cons

- missing some important commands. For example, there's no `goat exec` to invoke arbitrary scripts with the GOPATH. A convenience command to generate .go.yaml for an existing project would also be nice to compete with other dep managers.
- crosstalk between `loc` and `path` keys in .go.yaml. If you specify the `type` of a dependency to be `git`, then the `loc` is `https://github.com/foo/bar.git` and the `path` is `github.com/foo/bar`. So you end up having to write the same thing twice. Some magic defaults for this would be pretty sweet to reduce the amount of typing.
- format is missing some bells and whistles. But this is YAML, therefore very flexible. There's room to grow here if goat becomes popular.
- no svn/bzr support

## goop

###### Pros

- extremely bundler-like. rubyists will probably feel most at-home using goop since it has a Goopfile.lock, unlike every other bundler-like competitor.
- don't want to vendor the entire package in your project! Fine, just commit the Goopfile.lock instead like a rubyist would. goop will use the lock file if it exists, rather than recomputing your Goopfile.

###### Cons

- questionable longevity of Goopfile.lock. Ruby has a central repository to preserve old unmaintained gems, Go does not. Vendoring the entire package protects you against disappearing GitHub repositories, which could catch you off guard if your VCS only contains Goopfile.lock.
- Goopfile structure is simple but dense, and not standardized. Judicious application of whitespace in your Goopfile can avoid this issue (but still, data representation languages like YAML and JSON are more expressive).
- convenience command to generate Goopfile for an existing project would be nice to compete with other dep managers.
- no svn/bzr support
- `goop go test` is a bit more annoying than `goop test` (yes yes it's nitpicking)

## gopkg.in

###### Pros

- no extra tooling required, just the `go` tool you already know and love. This is a huge advantage for beginners. If you use gopkg.in, a newbie can `go get` your code and still have some version constraints enforced. In comparison, other tools fall flat on their faces if the user doesn't actually *have* the tool... which is fine for a developer of the package, but if I only want to use your widget, I probably don't want to install a dependency manager just for you. If the Go community settles on one tool to rule them all (ie if we find our Bundler), this advantage is lessened.
- enforcing major version only forces you (the consumer) to be aware of the backwards compatibility promises that your dependency offers, and to not rely on undocumented behavior. Conversely, there is also pressure on the supplier to uphold those constraints or else face the wrath of angry open source developers. Both of these things are good for everyone involved.

###### Cons

- only works with GitHub
- can't specify minor/patch versions. Running `go get` against a gopkg.in URL could yield two different packages, both of which satisfy the major version constraint but might be different minor/patch versions. Team has to be careful to keep their dependencies in sync, if they depend on behavior that was only introduced in a sub-version.
- have to change your imports. Apparently some people think this is sacrilegious. It's certainly an inconvenience (how major or minor depends on whom you ask).

## Roll your own

###### Pros

- you can do anything you want, everything you want, and nothing but what you want. Create a solution that exactly fits your project's needs. The only limitation is your skill with bash and your imagination (or whatever language you write your vendoring scripts in)
- some VCSes provide useful features for vendoring (eg git subtrees) that are not leveraged by any existing tool (as of writing). If you want these cool toys you have to do it yourself.

###### Cons

- you have to maintain your custom solution, which could be easy or could be a drag. Keeping track of all the dependencies in your project is unpleasant to do by hand, and it's code you have to write if you do it with the existing Go standard library.
- as the community converges on a preferred approach, your custom solution will begin to appear arcane in comparison. "Why do I have to use this weird shell script? Can't I just use `insert tool here`?" But you can always migrate your solution to what becomes standard, since they're all based on the GOPATH one way or another.
- your choice of tool is a message to the community. This is your vote on what Go dependency manager is "the one". Rolling your own solution is arguably throwing the vote away (or a vote of no confidence - in which case, why not contribute to the tool you like most, to make it suitable for your use case?).

# What would you do, @tummychow?

I'd gonna pretend dependency management isn't a problem `/buries head in the sand`

Just kidding. If you want to know what I think, just wait for me to make a large Go project. I'll put my money where my mouth is by actually *using* the tool I like. What a novel concept. Anyway my opinion is irrelevant. Evaluate the choices for yourself and be an informed customer!
