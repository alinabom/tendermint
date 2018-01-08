# Consensus Reactor

Consensus Reactor defines a reactor for the consensus service. When a peer p is added on the consensus reactor, it 
starts to manage peer state for the peer p, and starts the following three routines for the peer p: 
    - Gossip Data Routine
    - Gossip Votes Routine
    - QueryMaj23Routine.
    
## Gossip Data Routine

It is used to send the following messages to the peer: `BlockPartMessage`, `ProposalMessage` and 
`ProposalPOLMessage` on the `DataChannel`. The gossip data routine is based on the local round state (denoted rs) 
and the known round state of the peer (denotes prs) that are being continuously updated during the course of this routine.
The routine repeats forever the logic shown below:

```
1a) if rs.ProposalBlockPartsHeader == prs.ProposalBlockPartsHeader and the peer does not have all the proposal parts then
      Part = pick a random proposal block part the peer does not have 
      Send BlockPartMessage(Height, Round, Part) to the peer on the DataChannel 
      If send returns true, record that the peer knows the corresponding block Part
	  Continue  

TODO: We should also probably cover the case where rs.ProposalBlockPartsHeader != prs.ProposalBlockPartsHeader.  
For example, rs.Height == prs.Height and rs.Round > prs.Round. What is the right
thing to do in this case? Should we send parts to our peer from the previous round when we know that we have moved to 
the higher round? Consider also the case when rs.Height < prs.Height. Although we are lagging behind, we might be still 
receiving messages from the higher consensus instances from other peers that are helping me catchup so I should forward
these messages immediatelly to my peer. Similar reasoning applies when rs.Height == prs.Height and rs.Round < prs.Round.
 
1b) if (0 < prs.Height) and (prs.Height < rs.Height) then
      help peer catch up using gossipDataForCatchup function
      Continue

1c) if (rs.Height != prs.Height) or (rs.Round != prs.Round) then 
      Sleep PeerGossipSleepDuration
      Continue
TODO: Maybe remove this rule and define preciselly what is the scenario when we can't help our peer (if there is such
scenario and when we should sleep!   

//  at this point rs.Height == prs.Height and rs.Round == prs.Round
1d) if (rs.Proposal != nil and !prs.Proposal) then 
      Send ProposalMessage(rs.Proposal) to the peer
      If send returns true, record that the peer knows Proposal
	 if 0 <= rs.Proposal.POLRound then
	   polRound = rs.Proposal.POLRound 
       prevotesBitArray = rs.Votes.Prevotes(polRound).BitArray() 
       Send ProposalPOLMessage(rs.Height, polRound, prevotesBitArray)
    Continue  
TODO: We should probably forward Proposal and ProposalPOLMessages even when we are not at the same height and round
as our peer, for example if we are lagging behind. Otherwise we might be blocking those messages about reaching our peer
in a timely fashion which is critical for consensus termination.  

2)  Sleep PeerGossipSleepDuration 
TODO: Probably we should remove this rule as we can almost always do something useful for our peers. Would it make sense
to be more message driven at this level, i.e., upon receiving message to decide based on peer state if we should throw 
message or keep it and forward it.
```


 


 
 

    


    



  


