# pypeplot

Python pipe plotting utility. `pypeplot` provides a simple command-line
interface to plot data read from the standard input using `matplotlib`.
The intent to enable the quick generation of one-off plots to visualize
data, without necessarily exposing every `matplotlib` feature.

# Installation

`pyeplot` has no dependencies except for python and `matplotlib`.

The recommended installation method is through `pip`. You can install
the package globally as root, or install the package only for your user
in `~/.local/bin` (which may require changing your path).

    sudo pip install pypeplot

OR

    pip install --user pypeplot

If your system doesn't have pip available, you can install it as
follows:

    curl -sS https://bootstrap.pypa.io/get-pip.py | sudo python

If for some reason you don't want to use pypi (highly recommended), you
can download `pypeplot` manually, put it somewhere in your path, and
make it executable.

     curl https://raw.githubusercontent.com/acornejo/pypeplot/master/pypeplot/cmd.py | sudo tee /usr/bin/pypeplot >/dev/null
     sudo chmod 755 /usr/bin/pypeplot

# Usage

By default, `pypeplot` will read from standard input split each line
into columns and treat each column as a separate dataset to be plotted.
The default column separator is whitespace, but this can be changed to
any string through the `--delimiter` option, or the shorthand `-D`. Next
we provide an example to plot a csv file.

    cat file.csv | pypeplot -D ','

To deal with (potentially) endless inputs `pypeplot` uses a sliding
window. The default size of this window is 50 samples, but it can be
changed through the `--samples` option, or the shorthand `-S`. Setting
the number of samples used to zero will disable the sliding window and
have `pypeplot` render all data read from standard input.

    seq 100 | awk '{print sin($1/5)}' | pypeplot -S 0

Unless specified, `pypeplot` will attempt to plot every column in the
input, the indices of specific columns to be plotted can be provided
with `--column` option. The next snippet will plot only the zeroth and
second columns (which requires the input to have at least three
columns).

    seq 100 | awk '{print $i, $i*2, $i*3}' | pypeplot --columns 0 2

The x-value of each dataset is assumed to be implicitly encoded by the
line number in the input while the value of each column encodes the
y-value of each dataset. Sometimes it is desirable to encode the x-value
of the dataset in the columns as well (instead of the line numbers).
This can be specified by the `--xcolumn` option.

    seq 50 | awk '{print cos($1/8), sin($1/8)}' | pypeplot --xcolumn 0

`pyeplot` supports some basic options to format the generated plots,
including plot titles, axes labels, per-dataset legends and
per-dataset line styles and colors. Below we provide an example of how
to use these options.

    seq 50 | awk '{print $1*sin($1/8)/50, cos($1/8)/$1, sqrt($1/8)/5}' | pypeplot --title 'super important plot' --xlabel 'time' --ylabel 'speed' --legend we got labels --style 'bo' 'g^' 'r--'

Except for the style, most options should be self explanatory. In the
example above we set the style of the three curves to 'blue circles',
'green dots' and 'red dashes'. To read the style strings supported and
their meaning, see
[here](http://matplotlib.org/api/pyplot_api.html#matplotlib.pyplot.plot)

# Options

Option                        | Description                            | Default
------------------------------|----------------------------------------|---
--help, -h                    | Show help screen                       | n/a
--tee, -t                     | Copy standard input to standard output | off
--delimiter DEL               | String used as delimiter               | " "
--samples SAMPLES             | Number of sliding window samples       | 50
--columns COLUMN [COLUMN ...] | List of column indices to plot         | all
--xcolumn COLUMN              | Index of column to use as x-axis       | none
--title TITLE                 | Plot title                             | none
--xlabel                      | X-axis label                           | none
--ylabel                      | Y-axis label                           | none
--legend LEGEND [LEGEND ...]  | Legend to use for each plot            | none
--style  STYLE [STYLE ...]    | Style to use for each plot             | auto
--ymin YMIN                   | Minimum limit y-axis range             | auto
--ymax YMAX                   | Maximum limit y-axis range             | auto

# Examples

Plot the round-trip time to google.com as reported by ping

    ping google.com | awk -F'=| '  '/icmp/ {print 0;fflush}'  | pypeplot

Plot http request times to google.com using curl.

    while true; do curl -s -w "%{time_total}" -o /dev/null http://www.google.com; sleep 0.5; done | pypeplot

Plot the memory usage of a particular process

    while true; do ps -o rss -p $PID | awk '/[1-9]+/ {print $1;fflush}'; sleep 0.5; done | pypeplot

Plot the memory and CPU usage of a particular process

    while true; do ps -o rss,time -p $PID | awk '/[1-9]+/ {print $1, $2;fflush}'; sleep 0.5; done | pypeplot --legend memory cpu

Plot system memory available in Linux:

    while true; do free; sleep 0.5; done | awk '/Mem/ {print $4;fflush}' | pypeplot

Plot system memory available in OS X:

    while true; do vm_stat; sleep 0.5; done | awk '/free/ {print $3;fflush}' | pypeplot

*NOTE:* In the examples above which use awk we use the instruction
fflush to force awk to disable input buffering. This allows pypeplot to
receive the data and plot it as soon as it is produced (i.e. "live"
plots). This option is not stictly required, you could instead
have awk buffer the input, and then pypeplot would render the data when
awk flushes on its own (either when its buffer is full, or when the
stanrard input returns EOF).
