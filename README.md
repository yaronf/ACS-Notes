# ACS Notes

Review comments on the Agent Control Standard documentation. The comments are in `yaronf-review-notes.md`.

`upstream/` is a submodule: a pointer to the specification repository and one commit of it.

https://github.com/GenAI-Security-Project/agent-control-standard

Clone this repo with the spec included:

```bash
git clone --recurse-submodules https://github.com/yaronf/ACS-Notes.git
```

When the specification branch moves forward in a straight line, fast-forward the submodule and record the new commit here:

```bash
git submodule update --remote --merge
git add upstream
git commit -m "Fast-forward the upstream specification."
```
