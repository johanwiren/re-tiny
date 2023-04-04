# Re-tiny

I built re-tiny because I wanted something similar to re-frame when trying out ClojureDart.

Inspired by re-frame.

# Status

Highly experimental, useful in my hobby projects so far.

# How

Re-tiny is built using interceptors.

The default interceptors inject the following keys to the context map to mimic parts of re-frame

- `:db` - Your app state
- `:fx` - A collection of effects to be invoked when leaving
- `:dispatch` - A collection of events to dispatch when leaving

Both `fx` and `dispatch` should be `conj`'ed into when updating, don't overwrite them!

# Interceptors

```clojure
(def conj-1
  {:enter (fn [ctx] (update ctx :some-key conj :some-val))})

(def conj-2
  {:enter (fn [ctx] (update ctx :some-key conj :some-other-val))})

(rt/execute {:some-key []} [conj-1 conj-2]) ;; => {:some-key [:some-val :some-other-val]}
```

# Events

```clojure
(ns acme.main
  (:require [johanwiren.re-tiny.alpha :as rt]))

;; We specify events as a special interceptor that takes an extra argument
(def map-event
  {:name :add-user
   :enter (fn [ctx user]
            (update-in ctx [:db :users] (fnil conj []) user))})

;; Or just use a function
(def fn-event
  (fn [ctx user]
    (update-in ctx [:db :users] (fnil conj []) user)))

;; An effect (`:fx`) is a one-arity function
(def my-fx [arg]
  (println arg))

;; We can submit events
(rt/submit [map-event {:name "john"}])

;; We can inspect the db
@rt/db ;; => {:db {:users [{:name "john"}]}}
  ```

# Subscriptions

`cljd.flutter` provides `$` and `<!` that can subscribe to the db state

```clojure
(ns acme.subs
  (:require [cljd.flutter :as f]
            [johanwiren.re-tiny.alpha :as rt]))

;; We can subscribe to parts of our database
(def users (f/$ (-> rt/db f/<! :users)))
```

# Not working yet

- No `error` path in the interceptor chain
