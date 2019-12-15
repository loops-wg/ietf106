# IETF106 NWCRG Meeting, LOOPS slot

Time: Thursday 2019-11-21, approximately 15:10–15:30


## 07- About LOOPS (Local Optimizations on Path Segments) (Carsten Bormann)
<https://tools.ietf.org/html/draft-welzl-loops-gen-info>

Notes distilled from [NWCRG meeting
minutes](https://datatracker.ietf.org/meeting/106/materials/minutes-106-nwcrg)
and [video](https://youtu.be/WJ_Rq9SqnUc?t=6238).

Carsten Bormann asked who had been in a previous LOOPS side meeting,
about half of the people in the room (approximately 10 of 20) showed
their hands.  Carsten then gave a brief introduction to the LOOPS
activity.

Dave Oran asked: Is reordering one of the things you are not doing?
Carsten: The network might do reordering and we might have to do resequencing.
Dave: Is it a requirement to take packets misordered inside the tunnel
and fix the misordering at the tunnel exit?
Carsten: We want to be robust against reordering in the tunnel;
whether reordering needs to be fixed at the exit \[resequencing] we do not know yet
(Carsten mentioned that he would like to avoid having to do it).

Marie-Jose Montpetit: With or without chair hat: how can we collaborate?
Carsten: First, we look at outputs from the research group; also, based on a number
of assumptions, we might have specific questions where we would look
for help from the RG.
Marie-Jose: We indeed have strong opinions.
Carsten: I guess the WG asks specific questions, that may end up being
research questions, where we need research and simulations to answer them.
(Carsten then gave a question about good choices for the FEC
symbol size "E" that he had asked earlier in the meeting as an example.)

Emmanuel Lochin: There are many research issues here. With tunnels, you may aggregate flows and the protection may be per-flow or on the aggregate. I do not have the answer for that. You may have different types of flows and different coding schemes for them. You may end up having head of line blocking.
Carsten: This is about different traffic classes?
Emmanuel: ... and microflows, so you may have packets from different
flows, and a packet may be missing from one of the flows slowing down
another flow.
Carsten: We will have a hard time identifying the different flows and so
may not encounter this issue. Can you send the results you have to the list?
Emmanuel: Sure; we have some results with PIE and Codel.
Marie-Jose: If you do that, please copy both NWCRG and LOOPS lists.

Colin Perkins (without any hats on): You list a bunch of FEC schemes
with very different characteristics.  When we worked on video and
audio for AVT, we started with a simple scheme and went into more
complexity.  Do you expect the same?  You may end up having different
FEC solutions for different levels of protection, etc.
Carsten: We may want to dynamically change the FEC scheme that is used.
Marie-Jose: Add a specific signaling to detail the code that you want to use?
Carsten: Yes.
Marie-Jose: For QUIC, we added some signaling.
Colin: And we did for real-time media.
Carsten: Feedback information might include what packets were lost or
maybe just how many.
Colin: You may also have ACK mechanisms that report different things
depending on the FEC mechanism that is used.

Carsten then gave his view on the next steps for LOOPS: Explore the
design space, until a WG is formed — hopefully in time for IETF 107.
Marie-Jose: You may want the same type of interaction between NWCRG
and LOOPS as there is between T2TRG and CORE.
