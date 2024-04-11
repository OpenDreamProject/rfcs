# OpenDream RFC Repo

This repository stores RFCs for the [OpenDream](https://github.com/OpenDreamProject/OpenDream) project. Current and future users of OpenDream are strongly encouraged to watch this repository and provide constructive input on all open pull requests.

## Motivation

Although the core goal of OpenDream is to provide one-way parity with BYOND, there are [many opportunities](https://github.com/OpenDreamProject/OpenDream/wiki/Differences-Between-OpenDream-and-BYOND#enhancements) in which OpenDream and the DM language can be improved while still enabling users to migrate from BYOND with minimal effort. Or to put it another way, we can add new functionality *on top* of BYOND parity so that OpenDream users can take advantage of them while BYOND users are not impacted. 

A key example of this is [pragmas](https://github.com/OpenDreamProject/OpenDream/wiki/Pragmas---Error-Emissions). Pragmas provide configurable compiler emissions for things such as runtime errors that *can* be detected at compiletime but BYOND fails to detect. Utilizing pragmas while maintaining BYOND compatibility is as simple as declaring them in their own code file and putting this in your code somewhere:
```
#ifdef OPENDREAM
// Don't include the file of pragmas unless `OPENDREAM` is defined (which OpenDream defines for you)
#include("/path/to/pragma/file.dm")
#endif
```

Furthermore, OpenDream is intended to provide a solution for *everyone* and not any specific codebase. To prevent any one person from turning the project into their own new programming language, this RFC process exists to solicit input from the wider community on any major changes that would impact their current or future usage of OpenDream.

## RFC Considerations

Whenever new syntax is introduced, we need to ask:

- Is it BYOND compatible?
- Is it easy for machines and humans to parse?
- Does it create grammar ambiguities for current and future syntax?
- Is it stylistically coherent with the rest of the language?
- Does it present challenges with editor integration like autocomplete?

For changes in semantics, we should be asking:

- Is behavior easy to understand and non-surprising?
- Can it be implemented performantly today?
- Is it compatible with type checking and other forms of static analysis?

As the OpenDream userbase grows, reversing these decisions will only become more costly and have backwards compatibility implications.

## RFC Process

To open an RFC, a Pull Request must be opened which creates a new Markdown file in `docs/` folder. The RFCs should follow the template `TEMPLATE.md`, and should have a file name that matches the RFC's title.

### RFC Drafts
If you are in the early stages of designing the RFC and would like to solicit feedback on major considerations for your proposal, but it isn't ready for formal review, you can open your RFC as a Draft Pull Request. Once you are ready, mark it as ready for review and proceed to the below section:

### RFC Review & Finalization
Every open RFC will be open for at least two calendar weeks. This is to make sure that there is sufficient time to review the proposal and raise concerns or suggest improvements. The discussion points should be reflected on the PR comments; when discussion happens outside of the comment stream, the points salient to the RFC should be summarized as a followup.

The first week should be used to resolve any major technical or design problems raised by the OpenDream team, though everyone is encouraged to provide input. After the first week and any major problems are resolved, a Maintainer from the OpenDream team should share the RFC in the following places to solicit feedback from the wider community for the second week:
- `#codebase-announcements` channel in OpenDream's Discord.
- `#maintainerbus` channel in coderbus's Discord.
- `#coderbus` channel in coderbus's Discord.

When the initial comment period expires, the RFC can be merged if there's consensus that the change is important and that the details of the syntax/semantics presented are workable. The decision to merge the RFC is made by the OpenDream team.

If at any point the RFC undergoes substantial changes (anything more than spelling/formatting/wording), the two-week timer may (at OpenDream staff discretion) be reset to the time the last substantial commit was pushed.

In general, RFCs can also be updated after merging to make the language of the RFC more clear, but should not change their meaning. When a new feature is built on top of an existing feature that has an RFC, a new RFC should be created instead of editing an existing RFC. That being said, a superceded RFC should be edited to include a notice and a link to the new RFC.

When there's no consensus that the feature is broadly beneficial and can be implemented, an RFC will be closed. The decision to close the RFC is made by the OpenDream team.

Note that in some cases an RFC may be closed because we don't have sufficient data or believe that at this point in time, the stars do not line up sufficiently for this change to be worthwhile, but this doesn't mean that it may never be considered again; an RFC PR may be reopened if new data is available since the original discussion, or if the PR has changed substantially to address the core problems raised in the prior round.
