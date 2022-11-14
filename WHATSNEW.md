Version 1.0-alpha1 (unreleased)
===============================

Initial release - forked from tanoconsulting/ezmigrationbundle ver. 1.0 alpha 3, merging wizhippo/ibexa-migration-bundle

*Explanation of the 'aplha' tag:*

1. the codebase itself is fairly _stable_ and _complete_, as it is a fork of a project which had over 75 releases already
2. on the other hand, given that the underlying cms framework has evolved a lot, there might be bugs due to API changes
3. also, not all features of the underlying cms framework are fully supported

*BC guarantees on the road to 1.0 release:*

- the current yml migration format will be kept stable (possibly parts of it deprecated, though)
- the cli commands api will be kept stable
- the php classes and services might be heavily refactored. Please no subclassing / injecting them in your code for now
