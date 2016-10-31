# [mount](https://github.com/tolitius/mount/)

### 특징
  > Conceptually, Mount takes the approach of encapsulating stateful resources
using namespaces. This leads to a natural separation between the code that
deals with state from the pure core of the application business logic.

  > I think that it helps to treat the core business logic as you would a library. It
should be completely agnostic regarding where the data is coming from and
where it’s going. We should never pass resources directly to our business
logic. Instead, we should create a thin layer that deals with external resources
and calls the business logic to do the processing of the data.


### 기초
```clojure
(require '[mount.core :as mount])

(mount/defstate lester
  :start "hi"
  :stop "bye")
;"|| mounting... #'mount-example.core/lester"
;=> #'mount-example.core/lester

(mount/start)
;=> {:started ["#'mount-example.c/state" "#'mount-example.b/state" "#'mount-example.a/state"]}

(mount/stop)
;=> {:stopped ["#'mount-example.a/state" "#'mount-example.b/state" "#'mount-example.c/state"]}

```

### 좀 더
```clojure
(require '[mount.core :as mount])

(mount/start #'mount-example.a/state)
(mount/start-without #'mount-example.b/state)
(mount/start-with {#'mount-example.a/state "1"})
(mount/start-with-args {:name "lester"})

(mount/defstate ttest
  :start (do (println "ttest.hi")
            "11")
  :stop (do (println "ttest.bye")
            "ttest"))

(mount/start-with-states {#'mount-example.a/state #'ttest})

(mount/stop #'mount-example.a/state)
(mount/stop-except #'mount-example.a/state)

```

### 상태
```clojure
(require '[mount.tools.graph :as graph])

(graph/states-with-deps)
;=>
({:name "#'mount-example.b/state", :order 1, :status #{:stopped}, :deps #{}})
{:name "#'mount-example.a/state", :order 2, :status #{:stopped}, :deps #{}}

; @#'mount/running
;=>
; #object[clojure.lang.Atom
;         0x25fa85a7
;         {:status :ready,
;          :val {"#'mount-example.c/state" {:stop #object[mount_example.c$eval249$fn__252
;                                                         0x7eff91e7
;                                                         "mount_example.c$eval249$fn__252@7eff91e7",]}
;                "#'mount-example.b/state" {:stop #object[mount_example.b$eval266$fn__269
;                                                         0x473b5c28
;                                                         "mount_example.b$eval266$fn__269@473b5c28"]},
;                "#'mount-example.a/state" {:stop #object[mount_example.a$eval283$fn__286
;                                                         0x4f007efc
;                                                         "mount_example.a$eval283$fn__286@4f007efc"]}}}]

;; current master branch
(defn running-states []
  (keys @running))

(mount/running-states)
;=> ("#'mount-example.c/state" "#'mount-example.b/state" "#'mount-example.a/state")

```

### 기타
```clojure
^{:on-reload :noop}
^{:on-reload :stop}

(require '[mount.core :refer [only with-args except swap swap-states] :as mount])

(mount/only #{#'mount-example.a/state})
(mount/with-args {:name "lester"})
(mount/except #{#'mount-example.a/state})
(mount/swap {#'mount-example.a/state "1"})
(mount/swap-states {#'mount-example.a/state #'ttest})

(-> (only #{#'mount-example.a/state
            #'mount-example.b/state})
    (with-args {:name "lester"})
    (except #{#'mount-example.a/state})
    (swap {#'mount-example.b/state "1"})
    (swap-states {#'mount-example.a/state #'ttest})
    (mount/start))

```

자매품: [mount-lite](https://github.com/aroemers/mount-lite)
