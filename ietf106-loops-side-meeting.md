# IETF106 LOOPS Side Meeting

Time: Tuesday 2019-11-19 8:30-9:45, Room: Orchard

Carsten Bormann called the meeting to order and reminded everyone
about the rules on the note-well slide.  Agenda bashing and a quick
round of introductions followed.  Carsten asked who had attended a
previous LOOPS side meeting: about half of the attendees showed their
hand (about 25–30 were present; a few more people came in later).

Agenda after that:

- Carsten: quick recap of loops
<!-- (congestion feedback not the scope of this meeting) -->
- All: technical discussion, focusing on three items
- Carsten: next steps

## Carsten: recap on LOOPS

Carsten gave a quick overview of the LOOPS activity, how that is not
exacly a traditional transport protocol, how it relates to other
activities in IETF and IRTF, such as the recent drafts about Sliding
window FEC, and what drafts are already available about LOOPS.

Michael Welzl had contributed just-in-time some slides about recent
research, which managed to raise some discussion.
Gorry Fairhurst commented that the very high loss rates that can be
seen on the results slide are not realistic.
Michael explained that the slides show that packet loss needs to become
extremely high before the buffers start to fill.
Carsten added that the LOOPS implementation may want to limit the
overhead (i.e., the data it adds) way before the extreme loss rates
shown on the slide are reached, e.g., at 10 % of the payload data rate
maximum.

Yong Cui: what kind of tunnel will LOOPS use, and how will the
end-to-end transport layer’s congestion control interact with that of LOOPS?
Carsten: some end to end processing is always happening; LOOPS has to
make sure any end-to-end congestion control in that keeps working, by
giving signals to that that enable it to control its sending rate.
We do not want to damage the operation of the end-to-end congestion control.
Within LOOPS, could run its own congestion control, but don't have to
do this if relaying congestion information to the end-to-end
congestion control, which will control the sending rate for us.
We probably want to include a "circuit breaker" in case that doesn't work.
Yong Cui: interaction with underlay tunnel protocol -- discuss offline.

Gorry brought up the case of a QUIC session running over LOOPS.  So
LOOPS would have to drop packets to inform QUIC's congestion control.
Carsten replied that the congestion signal can be supplied both via
ECN and via dropping packets.
Gorry asked what is the point of repairing when you next need to drop
those packets.
Carsten surmised that a lower drop percentage might be able to
achieve the same effect on congestion control, by making use of loss
events.
Gorry is not convinced about that for the non-ECN case with QUIC; so
that will require further examination.

## All: Discussion items

### ACKs and retransmission

Carsten said that the specification needs to define what information
the two LOOPS nodes make available to each other and then how that is
encoded in a specific protocol.

With that, a retransmission decision can be a local decision, and,
like with TCP, with time, the algorithms we use for that will become
better, without having to change the protocol.  So we don't need to
decide on those algorithms, but we want to define the information set
in such a way that a good decision can be made, and we probably want
to supply a basic mode of operation if only as existence proof or as
guidance for implementers that want to implement something simple.
Carsten pointed to the traditional dupack style that (almost)
immediately reacts to holes in the acknowledgements sent back, as well
as the time-based approach that interprets the lack of an ACK after
some time (as in RACK).

The design of the information set: PSNs (packet sequence numbers) are
added, which number payload packets; an "ACK desirable" flag can
solicit the prompt return of a "block 1" ACK; "block 2" ACKs provide
some aggregate information.
So there are no "negative acknowledgements", just acknowledgements
with holes.

(There is also the end-to-end retransmission mechanism, which might
generate spurious retransmissions.)

#### Single vs. multiple retransmission

Carsten mentioned that a simple implementation strategy is to only
ever retransmit once; if multiple retransmissions are desired, the
implementation needs to have a way to stop attempting retransmission
at some point.
Yizhou Li asked the audience whether it had cases in mind that would need
multiple retransmissions; if we don't have any, we might focus on
single retransmission (potentially in a unified mechanism).
Yong suggested combining single retransmission with FEC for those cases that
need more help.
Gorry worried that performing multiple retransmissions might require
more sophisticated interaction with congestion control; a single
retransmission might fly under the radar most of the time.
Carsten pointed out that potentially multiple retransmissions at a low
overall loss rate might do less damage that single retransmissions at
a higher rate.
Gorry commented that it is quite difficult to know what will happen in
these cases.
Carsten pointed out that the threshold where things become difficult
may not exactly be at the boundary between single and multiple
retransmissions; he suggested to not exclude multiple retransmissions
at this time.
He pointed out that focusing on single retransmissions could be used
to simplify the protocol or just the implementations.
Gorry could also imagine much more complicated ways to perform
retransmissions.
Carsten suggested to use FEC for these cases, and Gorry agreed.
Michael pointed out that the ratio between local RTT and end-to-end RTO matters.
If ratio is small, can do multiple retransmissions.
But we do not know this ratio.
So as a WG, he recommneded to start with single first.
Yong said that there are different patterns of loss (e.g., burst
loss), and suggested choosing methods according to these.

#### Robustness against reordering

Carsten then asked whether LOOPS should be robust against reordering
(change of packet order) in the LOOPS path segment, and whether LOOPS
should resequence (restore the original packet order) at the end.
Michael opined that we should never resequence: let the end systems handle
the reordering.  Otherwise, additional latency will be created.

Carsten restated the question to focus on robustness against
reordering on the path segment.
Gorry cited recent discussions in QUIC that concluded the Internet was now
designed to avoid reordering beyond a threshold of three packets.

Carsten compared this situation to buses, where there are no seats, so
everybody knows they need to bring their own foldable chars for the bus
ride.  This is stable because buses don't need seats when people bring
foldable chairs anyway.  (This is similar to the argument that the
end-to-end protocol does not need to handle reordering because the
path does the work to take care any reordering is limited to three packets.)

Gorry said that this (moving the onus from the network to the end
systems) was great for research work, but for IETF activity, we should
better accommodate the current system.
Gorry asked three questions:

1. Do we want to be resilient to reordering (at small levels) --
   probably makes sense
2. The same at larger levels -- probably still makes sense
3. Change the network -- that will be controversial.

Gorry: people do have silicon in switches where they would benefit
from pulling out the no-reordering requirement.
Carsten: it is not changing the network, just preparing for future change...
Gorry: for IRTF, it is ok, but not for IETF, where it is dangerous.

Joe would like to support both a loss-based and a delay-based
upper-layer protocol.

Michael said that robustness to reordering is probably good; but
adding resequencing makes things potentially worse due to extra delay.
Joe explained the interaction between segment recovery and end-to-end
congestion control.

Carsten summarized his conclusion: We probably want to optimize for
networks with multiple packets in flight, and with loss rates that are
not "insane".  Loss detection can be completely hole-based ("dupack")
or time-based as in RACK.  So structure the information set provided
by the protocol so that becomes a local decision; enough information
is provided for both approaches.

### Encapsulation

Nobody focusing on encapsulation was in the room, so we skipped the
item in the interest of time.

### FEC discussion

Carsten opened the spectrum of FEC mechanisms, from very stupid ways
(just sending packets twice) via very simple, parity-based ones, to
more sophisticated ones.

Vincent Roca proposed not to use too simple FEC, and said there is
little benefit to use parity.
In QUIC, it was found that a simple XOR system is
not very useful, which caused the work on FEC in QUIC to stop; you may
use a simple FEC scheme but there are better schemes to use.
Carsten pointed out that we are not intending to provide 100%
reliability, so the results from the QUIC discussion may not quite apply here.
2D ("matrix") parity schemes like in SMTPE 2022 actually work.
Vincent still doesn't like them very much.
Carsten: we have 3 classes of FEC on the slide. May want to enlarge to more classes.
Yong: May enable FEC for retransmission; may use acknowledgements to
stop sending further repairing packets, ...
Carsten: I probably would not do it by stopping FEC
generation.  Dynamic mechanism with a control loop suggesting an FEC rate.
Chi-Jiun Su reminded us about IPR issues; and Carsten agreed that that’s important.
Vincent pointed out that some of this is now 20 years old so we do not
have to care too much about it.
Carsten suggested providing both encumbered and high-performance
variants; and asked to take the FEC scheme discussion to the mailing list.

Carsten presented a bit more about how FEC could be added to the
packets, assuming systematic codes.  There is an opportunity to
address MTU issues (as there already is heavy processing of the
packets).

## Next steps

The meeting ran out of time and Carsten summarized the next steps: We
do not have a Working Group yet, so we shouldn't make hard decisions;
but we can continue refining the documents on the mailing list as if
we were a WG.
Work on the WG charter can continue in parallel.

