# StatsCombiner

StatsCombiner is a Ruby gem for generating most-viewed widgets from the [Chartbeat API](http://chartbeat.pbworks.com/). Unlike most analytics systems, Chartbeat doesn't give you cumulative visitor counts. Rather, it takes live snapshots of people sitting on pages at a given time. StatsCombiner asks Chartbeat what these numbers are every `n` minutes during a given `ttl` and combines visitor counts where it finds matching `<title>`s, to allow popular stories to bubble up the list. When `ttl` expires, it will publish out a static HTML file with your top ten list to a location of your choosing, trash its database and start collecting again.

This gem is a rewrite from the PHP implementation I wrote about [here](http://blog.chartbeat.com/2009/08/04/guest-post-how-talking-points-memo-uses-chartbeat/).

## Installation

`gem install stats_combiner`

## Usage

Write a short combiner script that tells StatsCombiner your Chartbeat API parameters, how long it should combine for and where to put the flat file. Add filters to manipulate the data it will publish. 

Here's an example:

    require 'rubygems'
    require 'stats_combiner'
    
    # initialize the script
    s = StatsCombiner::Combiner.new({
      :ttl => 3600, #1 hour from first run
      :host => 'yourdomain.com',
      :api_key => 'YOURKEY',
      :flat_file => '/path/to/staticfile/top_ten.html'
    })
        
    # create a filters object and add some filters
    e = StatsCombiner::Filterer.new
    e.add :path_regex => /(\/$|\/index.php$)/, :exclude => true
    
    # run it!
    s.run({
      :filters => e.filters,
      :verbose => true  #this option reports combining status and TTL when true
     })

Then add this script to your crontab. I recommend running it every 5 minutes. Just be a good API-consumer when setting your cron:

    */5 * * * *       cd /path/to/my_combiner && ruby my_combiner.rb

### Filters

    http://{prefix}.{host}/{path}{suffix}

search by..

    :title_regex => regexp                # Filter on a title pattern
    :path_regex=> regexp                  # Filter on a path pattern

..to add a:

    :suffix => string or regexp           # a path modification
    :prefix => string                     # a subdomain
    :modify_title => bool or regexp       # Modify the title inline 
                                          # (true replaces title match with '')

..or, to ignore the entry:

    :exclude => bool                      # Exclude matching URLs from the top ten list
    
Some examples from TPM:
     
    e.add :prefix => 'tpmdc', :title_regex => /\| TPMDC/, :modify_title => true
    e.add :path_regex => /((\?|&)ref=.*)/, :suffix => ''
    e.add :path_regex => /(\?(page|img)=(.*)($|&))/, :suffix => '?\2=1'
    e.add :path_regex => /(\/$|\/index.php$)/, :exclude => true

### A note on testing

You may need to `sudo spec` if your user doesn't have write access to the gems directory.

## Author

Al Shaw (al@talkingpointsmemo.com)

## License (MIT)

Copyright (c) 2010 TPM Media LLC

Permission is hereby granted, free of charge, to any person obtaining a copy of this software and associated documentation files (the "Software"), to deal in the Software without restriction, including without limitation the rights to use, copy, modify, merge, publish, distribute, sublicense, and/or sell copies of the Software, and to permit persons to whom the Software is furnished to do so, subject to the following conditions:

The above copyright notice and this permission notice shall be included in all copies or substantial portions of the Software.

THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY, FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM, OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE SOFTWARE.