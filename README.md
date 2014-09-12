# clj-vw

A Clojure client and wrapper for [vowpal
wabbit](https://github.com/JohnLangford/vowpal_wabbit/wiki), a fast out-of-core learning system
sponsored by [Microsoft Research](http://research.microsoft.com/en-us/) and (previously) [Yahoo!
Research](http://research.yahoo.com/node/1914).

## Artifacts

clj-vw artifacts are released to Clojars.

If you are using Maven, add the following repository definition to your pom.xml:

```
<repository>
  <id>clojars.org</id>
  <url>http://clojars.org/repo</url>
</repository>
```

## The Most Recent Release

With Leiningen:

```
[engagor/clj-vw "1.0.0-RC3"]
```

With Maven:

```
<dependency>
  <groupId>engagor</groupId>
  <artifactId>clj-vw</artifactId>
  <version>1.0.0-RC3</version>
</dependency>
```

## Usage and documentation

Except when only using client related code for connecting to a remote vw daemon (see
[online.clj](src/clj_vw/online.clj)), this library requires that vowpal wabbit is installed locally.
Basic knowledge of vowpal wabbit, its [command line
options](https://github.com/JohnLangford/vowpal_wabbit/wiki/Command-line-arguments) and [input
format](https://github.com/JohnLangford/vowpal_wabbit/wiki/Input-format) are recommended. See the
[vowpal wabbit tutorial](https://github.com/JohnLangford/vowpal_wabbit/wiki/Tutorial) for more
information.

Codox documentation in html format can be generated by issuing the command `lein doc`. Documentation
files will be put under `.../clj-vw/doc/`. There are three namespaces.

### [clj-vw.core](src/clj_vw/core.clj)

Generic core functionality for interacting with vowpal wabbit (input example formatting, writing
data files, passing options and calling vw, ...)

All functions work on a `settings` argument, which is basically a map containing vowpal wabbit
options, training examples, etc. Options are set by calling `set-option`. Finally, vowpal wabbit is
called by calling the function `vw` on prepared settings.

#### Example:

```
(-> (set-option :data "foo/bar.dat"
                :final-regressor "foo/bar.model"
                :save-resume true
                :ngram 3
                :quadratic "ab"
                :learning-rate 0.3)
    (add-example {:label 1, :features ["a"]}
                 {:label -1, :features ["b"]})
    (write-data-file)
    (vw)
    (read-predictions))
```

#### Overview of available functions:

* `(available-options)`

  Print and return the set of available options (plus basic info).

* `(set-option settings key val & more)`

  Set one or more options. Can be chained, e.g.

  ```
  (def settings (-> {}
                    (set-option :data "foo/bar.dat")
                    (set-option :save-resume true)
                    (set-option :ngram 3)
                    (set-option :quadratic "ab")
                    (set-option :quadratic "ac")
                    (set-option :learning-rate 0.3)))
  ```

* `(get-option settings key)`

  Return option for `key` in `settings`.

* `(remove-option settings key & more)`

  Remove option `key` from `settings`.

* `(add-example settings example & more)`

  Add one ore more examples to settings.

* `(format-example {:keys [label labels importance tag features])`

  Turns an example given as a map into vowpal wabbit's string format (see
  https://github.com/JohnLangford/vowpal_wabbit/wiki/Input-format).

  ```
  (format-example {:label -1.231
                   :features ["f1"
                              {:name "f2" :value 3.5}
                              {:name "f3" :namespace "ns3"}]})
   => "-1.231 | f1 f2:3.5 |ns3:1.0 f3"
  ```

  See [clj-vw/core_test.clj](test/clj_vw/core_test.clj) suite for more examples.


* `(read-predictions settings)`

  Read vowpal wabbit prediction file `(get-option settings :predictions)`.

* `(write-data-file settings & {:as writer-settings})`

  Writes `(:data settings)`, a collection of examples, to `(get-option settings :data)`.

* `(vw-command settings)`

  Returns the vw command as a string as defined by `settings`. Useful for inspecting the command
  that will be issued when calling `vw` on `settings`.

* `(vw settings)`

  Calls vw as specified by `settings`. Puts output in `settings` under `:output` and returns
  updated settings.

  More info in [clj-vw/core_test.clj](test/clj_vw/core_test.clj).

### [clj-vw.offline](src/clj_vw/offline.clj)
    
Higher level helper functions for interfacing to a local vowpal wabbit install.

* `(predict settings)`

  Use an existing vowpal wabbit model (as specified by `(get-option settings :initial-regressor)` or
  `(get-option settings :final-regressor)`, in that order) to compute predictions for examples in a
  data file (as specified by `(get-option settings :data)`) or in memory (as specified by `(:data
  settings)`).

* `(train settings)`

  Train a vowpal wabbit model from a data file (as specified by `(get-option settings :data)`) or from
  a collection of in memory examples (as specified by `(:data settings)`).

### [clj-vw.online](src/clj_vw/online.clj)

  Higher level helper functions for launching and connecting to a (local or remote) vowpal wabbit
  running in daemon mode.

* `(daemon & {:as settings})`

  Start a vw daemon. Port (and any other options) can be set via vw options, e.g. `(vw-daemon
  (set-option :port 8003))`.

* `(connect settings)`

  Connect to a vw daemon. 

  Host is determined as the value of either `(get-in settings [:client :host])`, `(get-in
  settings [:daemon :host])` or `"localhost"`, in this order.

 Port is determined as the value of either `(get-in settings [:client :port])`, `(get-in
  settings [:daemon :port])`, `(get-option settings :port)` or `26542`, in this order.

  Example, to start a local daemon on port 8003 and connect to it, do:

  ```
  (-> (set-option :port 8003) 
      (vw-daemon) 
      (connect)).
  ```

* `(train settings)`

  Send examples to a vw daemon connection (as returned by `connect`) for training. Examples in the
  return settings (under `:data`) are extended with a `:prediction` slot corresponding to vowpal
  wabbit's prediction before training.

* `(save settings)`

  Save daemon's model to `(get-opt settings :final-regressor)`.

* `(predict settings)`

  Send examples to a vw daemon connection for prediction (without training). Predictions are put under
  `:predictions` in `settings`.

* `(close settings)`

  Close daemon and/or client and cleanup.

## Testing

On the command line, issue `lein test`. Requires that vowpal wabbit is installed and that the
directory `"/tmp/"` is read/writable.

## License

Copyright © 2014 Engagor

Distributed under the BSD Clause-2 License as distributed in the file [LICENSE](LICENSE) at
the root of this repository.
