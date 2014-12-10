# tideline
### (and zipline)



Jana Beck (@jebeck)
#### Software Engineer, Tidepool

(Tidepool is an open source, not-for-profit effort to build an open data platform and better applications that reduce the burden of type 1 diabetes.)


!['Tidepool logo'](images/tidepool_logo.png 'Tidepool logo')

On the Web:

- [Main](tidepool.org 'Tidepool.org')
- [Developer Portal](http://developer.tidepool.io/ 'Tidepool Developer Portal')
- [On GitHub](https://github.com/tidepool-org/)


!['The problem'](images/the_problem.png 'The problem')


!['The Tidepool platform'](images/tidepool_platform.png 'The Tidepool platform')



# Background


## Type 1 Diabetes

- autoimmune disease
- destruction of insulin-producing cells (β-cells) in the pancreas
- replacement of insulin by injection or infusion


## The Problem

- insulin is a dangerous drug
- dosing is difficult
- 100s of decisions a day


## Data!

Minimum 3 streams:

- insulin doses
- blood glucose checks
- carbohydrate intake


## Even more data!


### Insulin pump

- basal insulin
- bolus insulin


### Blood glucose meters

possibly many


### Continuous glucose monitor

blood glucose values every 5 min.


### Contextual data

- fitness tracker
- sleep
- calendar
- notes/annotations
- menstrual cycle
- other health data
- etc.



# Step One

- put all the data in the same place
- plot it on a single time dimension



# Step Two

[scroll](http://janabeck.com/tideline/example/current-progress.html)!


## Strategy

- 100% virtual scrolling

- SVG dims = HTML container dims

- d3's [zoom behavior](https://github.com/mbostock/d3/wiki/Zoom-Behavior 'd3: Zoom Behavior')

  + with zooming disabled

  + only using pan

- positioning done through `transform` attributes: `translate(500,0)` etc.


### ...plus

- a scrollbar hacked together using d3's [drag behavior](https://github.com/mbostock/d3/wiki/Drag-Behavior 'd3: Drag Behavior')


### ...but

!['cone of shame GIF'](images/cone_of_shame.gif 'cone of shame')

- the scroll thumb is a fixed size

- and there's no two-finger trackpad scrolling!


#### not to mention...

the performance leaves something to be desired

!['Han Solo oops GIF'](images/fail_solo.gif 'Fail Solo')



# Deep Dive #1

[tideline-style virtual scrolling](http://bl.ocks.org/jebeck/1974647d476b67a0439d)



## the structure of tideline


### top-level: interface-defining modules, e.g.:

- [oneday](https://github.com/tidepool-org/tideline/blob/master/js/oneday.js)
- [twoweek](https://github.com/tidepool-org/tideline/blob/master/js/twoweek.js)

(lots o' method chaining)


### next: datatype-specific plot modules

- [js/plot/](https://github.com/tidepool-org/tideline/tree/master/js/plot)
- roughly follow [Towards Reusable Charts](http://bost.ocks.org/mike/chart/ 'Mike Bostock: Towards Reusable Charts')


"To sum up:

implement charts as closures with getter-setter methods."

@mbostock


### making a chart

- write a chart factory module
- ordering of tasks is sometimes crucial!


### ...but

- crucial ordering is confusing
- even for the developer that designed the interface


### ...furthermore:

- crucial ordering ≈ lots of implicit local state
- difficult to iterate on (makes too many assumptions)
- can be bug-prone


!['I don't know what I expected](images/dunno_what_I_expected.gif 'I don't know what I expected Arrested Development GIF')



## so far: 2 problems

1. scrolling performance
1. tricky chart configuration



## scrolling performance: desiderata

- two-finger trackpad scroll
- smoother/faster framerate


### example #1

[current tideline](http://janabeck.com/tideline/example/bg-only.html 'tideline: just blood glucose'), blood glucose data only


- framerate drops into teens when using scrollbar
- noticeably sluggish, stuttering


### native scrolling

- abandon 100% virtual rendering strategy
- render a really *really* wide SVG


### example #2

[zipline, render everything](http://janabeck.com/zipline/example/no-filter.html 'zipline: NoFilter')

!['loading...'](images/imessage-loading.gif 'loading GIF...')


### combo approach

- "literal" rendering of SVG
- on-the-fly (virtual) rendering for data


### example #3

[zipline, filter data on scroll](http://janabeck.com/zipline/example/basic-filter.html 'zipline: BasicFilter')


### example #4

[zipline, data filtering triggered by crossing a date boundary](http://janabeck.com/zipline/example/date-trigger-filter-no-reuse.html 'zipline: DateTriggerFilter, no reuse')


### anything else to be done?

"The browser was not designed to be constantly creating and throwing out DOM nodes."

(Pete Hunt)


### virtual scrolling and reusing nodes

- [longscroll (@jasondavies)](https://github.com/d3/d3-plugins/tree/master/longscroll 'd3 plugins: longscroll')
- [Bill White on virtual scrolling](http://www.billdwhite.com/wordpress/2014/05/17/d3-scalability-virtual-scrolling-for-large-visualizations/ 'Bill White on virtual scrolling')
- [long scrolling image grid (@gmaclennan)](http://bl.ocks.org/gmaclennan/11130600 'long scrolling image grid')


### my implementation

[short and sweet](https://github.com/jebeck/zipline/blob/master/src/util/reusenodes.js 'zipline: reusenodes.js')


### final example

[zipline with date trigger filtering and node reuse](http://janabeck.com/zipline/example/date-trigger-filter-with-reuse.html 'zipline: DateTriggerFilter, with node reuse')



# Deep Dive #2
[zipline-style virtual rendering on scroll](http://bl.ocks.org/jebeck/5ffdaad2094499997a21 'zipline-style virtual rendering on scroll')



## Hat tip

fellow Tidepooler [Nicolas Hery](http://nicolashery.com/ 'Nicolas Hery')

- [Integrating D3 and React](http://nicolashery.com/integrating-d3js-visualizations-in-a-react-app/ 'Integrating D3.js visualizations in a React app')
- [mini-spot](https://github.com/nicolashery/mini-spot 'A small example web app combining React, Mori, and Flux-like patterns')



## chart configuration: desiderata

- declarative, not procedural
- all state is explicit (and global)
- iteration is easy (don't assume!)


### defining charts through data

focus on *what* is in the chart

**not**

*how* to build the chart


### inspiration

!['React](images/react.png 'React: A JavaScript library for building user interfaces')

[great intro video](https://www.youtube.com/watch?v=nYkdrAPrdcw 'Rethinking web app development at Facebook')


### secondary inspiration

!['webpack'](images/what-is-webpack.png 'webpack module builder')

compare [before](https://github.com/tidepool-org/blip/blob/eb27605cc56b84d0ea5550db2c9073c84ef7f295/gulpfile.js) and [after](https://github.com/tidepool-org/blip/blob/master/webpack.config.js)


### zipline chart configuration

chart configuration is just data: an array

[for example](https://github.com/jebeck/zipline/blob/master/example/daily.config.js 'zipline daily chart config')



# what's next

- experimenting with [Flux](http://facebook.github.io/flux/docs/overview.html 'Flux') for controlling chart state
- testing, testing, testing