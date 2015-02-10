_ = require("lodash")
jade = require("jade")
fs = require("fs")
glob = require("glob")
mm = require("marky-mark")
slug = require("slug")
markdownpdf = require("markdown-pdf")

jade_opts =
  pretty: true

universe = ->
  blog_entries = mm.parseMatchesSync("../../", [ "_src/blog_entries/*.md" ])
  _.each blog_entries, (page) ->
    page.url = "/blog/" + slug(page.meta.title) + ".html"
  return {"blog_entries":blog_entries}

task 'build.resume.html', (options) ->
  fs.writeFile "./tmp/resume.html", jade.renderFile("./_src/resume_layout.jade", _.merge(jade_opts, universe(), {page: mm.parseFileSync("./_src/resume.md")} )), (err) ->
    if err
      console.log err
    else
      console.log "The file was saved!"
    return

task 'build.resume.pdf', (options) ->
  markdownpdf().from('_src/resume.md').to './tmp/resume.pdf', ->
    console.log 'Done'
    return

task 'build.blogs', (options) ->
  _.forEach universe().blog_entries, (blog_entry) ->
    console.log(blog_entry.url)
    fs.writeFile('./tmp' + blog_entry.url, jade.renderFile('./_src/blog_entry_layout.jade', _.merge(jade_opts, universe(), {page: blog_entry})))
    (err) ->
      if err
        console.log err
      else
        console.log 'The file was saved!'
      return
    return
