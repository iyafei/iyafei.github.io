_ = require("lodash")
jade = require("jade")
filendir = require('filendir')
fs = require("fs")
fs_extra = require('fs-extra')
glob = require("glob")
mm = require("marky-mark")
slug = require("slug")
markdownpdf = require("markdown-pdf")
sstatic = require('node-static')
http = require('http')
memoize = require('memoizee')
file_concat = require('concat')
path = require('path')

jade_opts =
  pretty: true

universe = ->
  console.log("building the universe...")

  return {"blog_entries":_.map(glob.sync("_src/blog_entries/*"), (page) ->
    m = mm.parseFileSync(page + "/index.md")
    m.dest = "/blog/" + slug(m.meta.title) + "/index.html"
    m.url = "/blog/" + slug(m.meta.title)
    m.src = page

    m.assets = {"jpgs": glob.sync("#{m.src }/*.jpg")}

    # filters and replaces instances of local assets with absolute paths
    _.each m.assets.jpgs, (jpg) ->
      m.content = m.content.replace(path.basename(jpg), m.url + "/" + path.basename(jpg))
    return m
  ), "package": require("./package.json")}

memo_universe = memoize(universe);

task 'new.blog_entry', (options) ->
  maxPlusOne = Math.max.apply(null, _.map(glob.sync('_src/blog_entries/*'), (page) ->
    parseInt(path.basename(page, '.md'))
  )) + 1

  newFile = "./_src/blog_entries/#{maxPlusOne}/index.md"
  fs.writeFile newFile, """
  ---
  title: CHANGE ME
  publishedAt: #{new Date().toString()}
  ---
  """, (err) ->
    if err
      console.log err
    else
      console.log(newFile)
    return


task 'build.index', (options) ->
  fs.writeFile "./index.html", jade.renderFile("./_src/index.jade", _.merge(jade_opts, memo_universe() )), (err) ->
    if err
      console.log err
    else
      console.log "index.html"
    return

task 'build.blogs', (options) ->
  _.forEach memo_universe().blog_entries, (blog_entry) ->
    console.log(blog_entry.dest)
    fs.writeFile('.' + blog_entry.dest, jade.renderFile('./_src/blog_entry_layout.jade', _.merge(jade_opts, memo_universe(), {entry: blog_entry})))
    (err) ->
      if err
        console.log err
      else
        console.log 'The file was saved!'
      return
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
    console.log(blog_entry.dest)
    filendir.wa('.' + blog_entry.dest, jade.renderFile('./_src/blog_entry_layout.jade', _.merge(jade_opts, memo_universe(), {entry: blog_entry})))
    (err) ->
      if err
        console.log err
      else
        console.log 'The file was saved!'
      return
    return

task 'build.assets.style', (options) ->
  file_concat [
    './_src/style.css'
    './node_modules/normalize.css/normalize.css'
  ], 'style.css', (error) ->
    console.log('style.css')
    return

task 'build.assets.image', (options) ->
  _.forEach memo_universe().blog_entries, (blog_entry) ->
    _.each blog_entry.assets.jpgs, (jpg) ->
      dest = ".#{blog_entry.url}/#{path.basename(jpg)}"
      fs_extra.copy jpg, dest, (err) ->
        if (err)
          return console.error(err)
        console.log("#{jpg} -> #{dest}")

task 'build', (options) ->
  invoke('build.index')
  invoke('build.blogs')
  invoke('build.resume.pdf')
  invoke('build.resume.html')
  invoke('build.assets.style')
  invoke('build.assets.image')

task 'server', (options) ->
  console.log('server now running on port 8080...')
  file = new (sstatic.Server)('.')
  http.createServer((req, res) ->
    file.serve req, res
    return
  ).listen 8080
