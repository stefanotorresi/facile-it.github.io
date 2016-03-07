## Blog beta

### Cloning instructions
```
git clone --recursive git@github.com:Jean85/blog-beta.git
```
OR 
```
git clone git@github.com:Jean85/blog-beta.git
cd blog-beta
git submodule init && git submodule update
```

### Usage
 * To serve the site locally: `./hugo server -t casper --buildDrafts`
 * To generate a new post: `./hugo new -t casper post/2016-03-title.md`
