_ = require("lodash")
jade = require("jade")
fs = require("fs")
glob = require("glob")
mm = require("marky-mark")
slug = require("slug")
markdownpdf = require("markdown-pdf")
sstatic = require('node-static')
http = require('http')
memoize = require('memoizee')
file_concat = require('concat')

jade_opts =
  pretty: true

universe = ->
  console.log("building the universe...")
  blog_entries = mm.parseMatchesSync("../../", [ "_src/blog_entries/*.md" ])
  _.each blog_entries, (page) ->
    page.url = "/blog/" + slug(page.meta.title) + ".html"
  return {"blog_entries":blog_entries}

memo_universe = memoize(universe);

task 'build.resume.html', (options) ->
  fs.writeFile "./tmp/resume.html", jade.renderFile("./_src/resume_layout.jade", _.merge(jade_opts, memo_universe(), {page: mm.parseFileSync("./_src/resume.md")} )), (err) ->
    if err
      console.log err
    else
      console.log "The file was saved!"
    return

task 'build.resume.pdf', (options) ->
  markdownpdf().from('_src/resume.md').to './tmp/resume.pdf', ->
    console.log "resume.pdf"

task 'build.index', (options) ->
  fs.writeFile "./index.html", jade.renderFile("./_src/index.jade", _.merge(jade_opts, memo_universe() )), (err) ->
    if err
      console.log err
    else
      console.log "index.html"
    return

task 'build.resume.html', (options) ->
  fs.writeFile "./resume.html", jade.renderFile("./_src/resume_layout.jade", _.merge(jade_opts, memo_universe(), {page: mm.parseFileSync("./_src/resume.md")} )), (err) ->
    if err
      console.log err
    else
      console.log "resume.html"
    return

task 'build.resume.pdf', (options) ->
  markdownpdf().from('_src/resume.md').to './resume.pdf', ->
    console.log 'resume.pdf'
    return

task 'build.blogs', (options) ->
  _.forEach memo_universe().blog_entries, (blog_entry) ->
    console.log(blog_entry.url)
    fs.writeFile('.' + blog_entry.url, jade.renderFile('./_src/blog_entry_layout.jade', _.merge(jade_opts, memo_universe(), {entry: blog_entry})))
    (err) ->
      if err
        console.log err
      else
        console.log 'The file was saved!'
      return
    return

task 'build.assets', (options) ->
  file_concat [
    './_src/style.css'
    './node_modules/normalize.css/normalize.css'
  ], 'style.css', (error) ->
    console.log('style.css')
    return

task 'server', (options) ->
  console.log('server now running on port 8080...')
  file = new (sstatic.Server)('.')
  http.createServer((req, res) ->
    file.serve req, res
    return
  ).listen 8080

task 'build', (options) ->
  invoke('build.index')
  invoke('build.blogs')
  invoke('build.resume.pdf')
  invoke('build.resume.html')
  invoke('build.assets')
