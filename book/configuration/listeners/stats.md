Statistics {#config_listener_stats}
==========

Listener
--------

Every listener has a statistics tree rooted at *listener.\<address\>.*
with the following statistics:

  ---------------------------------------------------------------------------------
  Name                        Type              Description
  --------------------------- ----------------- -----------------------------------
  downstream_cx_total         Counter           Total connections

  downstream_cx_destroy       Counter           Total destroyed connections

  downstream_cx_active        Gauge             Total active connections

  downstream_cx_length_ms     Histogram         Connection length milliseconds

  downstream_cx_overflow      Counter           Total connections rejected due to
                                                enforcement of listener connection
                                                limit

  downstream_pre_cx_timeout   Counter           Sockets that timed out during
                                                listener filter processing

  downstream_pre_cx_active    Gauge             Sockets currently undergoing
                                                listener filter processing

  global_cx_overflow          Counter           Total connections rejected due to
                                                enforecement of the global
                                                connection limit

  no_filter_chain_match       Counter           Total connections that didn\'t
                                                match any filter chain

  ssl.connection_error        Counter           Total TLS connection errors not
                                                including failed certificate
                                                verifications

  ssl.handshake               Counter           Total successful TLS connection
                                                handshakes

  ssl.session_reused          Counter           Total successful TLS session
                                                resumptions

  ssl.no_certificate          Counter           Total successful TLS connections
                                                with no client certificate

  ssl.fail_verify_no_cert     Counter           Total TLS connections that failed
                                                because of missing client
                                                certificate

  ssl.fail_verify_error       Counter           Total TLS connections that failed
                                                CA verification

  ssl.fail_verify_san         Counter           Total TLS connections that failed
                                                SAN verification

  ssl.fail_verify_cert_hash   Counter           Total TLS connections that failed
                                                certificate pinning verification

  ssl.ocsp_staple_failed      Counter           Total TLS connections that failed
                                                compliance with the OCSP policy

  ssl.ocsp_staple_omitted     Counter           Total TLS connections that
                                                succeeded without stapling an OCSP
                                                response

  ssl.ocsp_staple_responses   Counter           Total TLS connections where a valid
                                                OCSP response was available
                                                (irrespective of whether the client
                                                requested stapling)

  ssl.ocsp_staple_requests    Counter           Total TLS connections where the
                                                client requested an OCSP staple

  ssl.ciphers.\<cipher\>      Counter           Total successful TLS connections
                                                that used cipher \<cipher\>

  ssl.curves.\<curve\>        Counter           Total successful TLS connections
                                                that used ECDHE curve \<curve\>

  ssl.sigalgs.\<sigalg\>      Counter           Total successful TLS connections
                                                that used signature algorithm
                                                \<sigalg\>

  ssl.versions.\<version\>    Counter           Total successful TLS connections
                                                that used protocol version
                                                \<version\>
  ---------------------------------------------------------------------------------

Per-handler Listener Stats {#config_listener_stats_per_handler}
--------------------------

Every listener additionally has a statistics tree rooted at
*listener.\<address\>.\<handler\>.* which contains *per-handler*
statistics. As described in the
`threading model <arch_overview_threading>`{.interpreted-text
role="ref"} documentation, Envoy has a threading model which includes
the *main thread* as well as a number of *worker threads* which are
controlled by the `--concurrency`{.interpreted-text role="option"}
option. Along these lines, *\<handler\>* is equal to *main_thread*,
*worker_0*, *worker_1*, etc. These statistics can be used to look for
per-handler/worker imbalance on either accepted or active connections.

  ----------------------------------------------------------------------------
  Name                   Type              Description
  ---------------------- ----------------- -----------------------------------
  downstream_cx_total    Counter           Total connections on this handler.

  downstream_cx_active   Gauge             Total active connections on this
                                           handler.
  ----------------------------------------------------------------------------

Listener manager {#config_listener_manager_stats}
----------------

The listener manager has a statistics tree rooted at *listener_manager.*
with the following statistics. Any `:` character in the stats name is
replaced with `_`.

  ------------------------------------------------------------------------------------
  Name                           Type              Description
  ------------------------------ ----------------- -----------------------------------
  listener_added                 Counter           Total listeners added (either via
                                                   static config or LDS).

  listener_modified              Counter           Total listeners modified (via LDS).

  listener_removed               Counter           Total listeners removed (via LDS).

  listener_stopped               Counter           Total listeners stopped.

  listener_create_success        Counter           Total listener objects successfully
                                                   added to workers.

  listener_create_failure        Counter           Total failed listener object
                                                   additions to workers.

  listener_in_place_updated      Counter           Total listener objects created to
                                                   execute filter chain update path.

  total_filter_chains_draining   Gauge             Number of currently draining filter
                                                   chains.

  total_listeners_warming        Gauge             Number of currently warming
                                                   listeners.

  total_listeners_active         Gauge             Number of currently active
                                                   listeners.

  total_listeners_draining       Gauge             Number of currently draining
                                                   listeners.

  workers_started                Gauge             A boolean (1 if started and 0
                                                   otherwise) that indicates whether
                                                   listeners have been initialized on
                                                   workers.
  ------------------------------------------------------------------------------------
