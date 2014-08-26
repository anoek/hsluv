fs  = require 'fs'
Buffer = require('buffer').Buffer

{exec, spawn} = require 'child_process'

require 'coffee-script/register'


task 'docker:build', 'Build Docker container for running tasks', ->
  args = ['build', '-t', 'husl-dev-environment', '.']
  console.log 'RUNNING docker ' + args.join(' ') + '\n'
  spawn 'docker', args, {stdio: 'inherit'}

task 'docker:run', 'Run Docker container', ->
  args = [
    'run', '-i', '-t'
    '-v', __dirname + ':/husl'
    'husl-dev-environment', '/bin/bash'
  ]
  console.log 'RUNNING docker ' + args.join(' ')
  spawn 'docker', args, {stdio: 'inherit'}

# These cake tasks are optional
do ->

  try
    colorspaces = require 'colorspaces'
    onecolor = require 'onecolor'
    eco = require 'eco'
    husl = require 'husl'
    {Png} = require 'png'
  catch error
    return
    
  task 'build:docs-images', 'Generate images', ->
    console.log "Generating demo images:"
    try
      fs.mkdirSync 'dist/img/demo'

    hslToRgb = (h, s, l) ->
      h *= 360
      s *= 100
      l *= 100
      c = onecolor "hsl(#{h}, #{s}, #{l})"
      [c.red(), c.green(), c.blue()]

    makeImage = (file, func, width, height) ->
      rgb = new Buffer width * height * 3
      for y in [0..height - 1]
        for x in [0..width - 1]
          pos = (y * width + x) * 3
          rgbVal = func x / (width - 1), y / (height - 1)
          rgb[pos + 0] = rgbVal[0]
          rgb[pos + 1] = rgbVal[1]
          rgb[pos + 2] = rgbVal[2]
      png = new Png rgb, width, height, 'rgb'
      png_image = png.encodeSync()
      fs.writeFileSync file, png_image.toString('binary'), 'binary'

    chromaDemo = (color) ->
      C = color.as('CIELCHuv')[1] * 0.8
      red = colorspaces.make_color 'CIELCHuv', [50, C, 10]
      return red.as 'sRGB'

    makeDemo = (name, func, width = 360, height = 200) ->
      console.log " - #{name}"
      file = "dist/img/demo/#{name}.png"
      func2 = (x, y) ->
        try
          husl._rgbPrepare func x, y
        catch e
          console.log x, y
          console.log x * 360, 100 - y * 100
          console.log func x, y
          console.log husl.p.toRGB x * 360, 100 - y * 100, 50
          throw e
      makeImage file, func2, width, height

    makeDemo 'husl', (x, y) ->
      rgb = husl.toRGB x * 360, 100 - y * 100, 50
      return rgb

    makeDemo 'huslp', (x, y) ->
      rgb = husl.p.toRGB x * 360, 100 - y * 100, 50
      return rgb

    makeDemo 'husl-chroma', (x, y) ->
      rgb = husl.toRGB x * 360, 100 - y * 100, 50
      return chromaDemo colorspaces.make_color 'sRGB', rgb

    makeDemo 'husl-low', ((x, y) ->
      rgb = husl.toRGB x * 360, 100 - y * 100, 10
      return rgb), 270, 150

    makeDemo 'husl-high', ((x, y) ->
      rgb = husl.toRGB x * 360, 100 - y * 100, 95
      return rgb), 270, 150

    makeDemo 'cielchuv', (x, y) ->
      color = colorspaces.make_color 'CIELCHuv', [50, 200 - y * 200, x * 360]
      if !color.is_displayable()
        rgb = [0, 0, 0]
      else
        rgb = color.as 'sRGB'
      return rgb

    makeDemo 'hsl', (x, y) ->
      return hslToRgb x, 1 - y, 0.5

    makeDemo 'hsl-lightness', (x, y) ->
      rgb = hslToRgb x, 1 - y, 0.5
      color = colorspaces.make_color 'sRGB', rgb
      l = color.as('CIELUV')[0] / 100
      return [l, l, l]

    makeDemo 'hsl-chroma', (x, y) ->
      rgb = hslToRgb x, 1 - y, 0.5
      return chromaDemo colorspaces.make_color 'sRGB', rgb

    makeDemo 'huslp-chroma', (x, y) ->
      rgb = husl.p.toRGB x * 360, 100 - y * 100, 50
      return chromaDemo colorspaces.make_color 'sRGB', rgb

  task 'build:docs', 'Build docs', ->
    console.log "Building index.html"
    template = (fs.readFileSync 'src/index.eco').toString()
    fs.writeFile 'dist/index.html', eco.render template,
      version: '4.0.2'

    console.log "Building js/main.js"
    exec "coffee --compile --output dist/js src/main.coffee", (err, stdout, stderr) ->
      throw err if err
      invoke 'build:docs-images'
