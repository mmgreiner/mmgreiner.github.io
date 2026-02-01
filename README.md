# mmgreiner.github.io

This is a collection of posts concerning programming, math, etc. It was originally created in [Jekyll](https://jekyllrb.com/), and later moved to [Hugo](https://gohugo.io).

The static generated pages are found on github pages [mmgreiner.github.io][mmgreiner-github].

[mmgreiner-github]: https://mmgreiner.github.io

## Development

During development, use:

~~~
hugo server
~~~


To generate them locally including the drafts (`-D`), use:

~~~
hugo -D --destination ~/Sites/mypages --baseURL http://localhost/~mmgreiner/mypages
~~~

You can browse it at <http://localhost/~mmgreiner/mypages>

## Production

To prepre for production, use `minify`:

```
hugo --minify
Start building sites … 
hugo v0.155.1+extended+withdeploy darwin/arm64 BuildDate=2026-01-30T10:10:48Z VendorInfo=Homebrew


                  │ EN  
──────────────────┼─────
 Pages            │ 182 
 Paginator pages  │   9 
 Non-page files   │   0 
 Static files     │  10 
 Processed images │   0 
 Aliases          │  68 
 Cleaned          │   0 

Total in 161 ms
```

Now, `./public` containsn all the necessary files.

With the workflow defined in  `.github/workflows/hugo.yaml`, upon committing and pushing the files, it should automatically be available on [mmgreiner.github.io][mmgreiner-github].



