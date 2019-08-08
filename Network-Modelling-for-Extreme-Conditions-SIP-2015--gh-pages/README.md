//Aodv Code
/* -*- Mode:C++; c-file-style:"gnu"; indent-tabs-mode:nil; -*- */
    2 /*
    3  * Copyright (c) 2009 IITP RAS
    4  *
    5  * This program is free software; you can redistribute it and/or modify
    6  * it under the terms of the GNU General Public License version 2 as
    7  * published by the Free Software Foundation;
    8  *
    9  * This program is distributed in the hope that it will be useful,
   10  * but WITHOUT ANY WARRANTY; without even the implied warranty of
   11  * MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE.  See the
   12  * GNU General Public License for more details.
   13  *
   14  * You should have received a copy of the GNU General Public License
   15  * along with this program; if not, write to the Free Software
   16  * Foundation, Inc., 59 Temple Place, Suite 330, Boston, MA  02111-1307  USA
   17  *
   18  * Based on 
   19  *      NS-2 AODV model developed by the CMU/MONARCH group and optimized and
   20  *      tuned by Samir Das and Mahesh Marina, University of Cincinnati;
   21  * 
   22  *      AODV-UU implementation by Erik Nordström of Uppsala University
   23  *      http://core.it.uu.se/core/index.php/AODV-UU
   24  *
   25  * Authors: Elena Buchatskaia <borovkovaes@iitp.ru>
   26  *          Pavel Boyko <boyko@iitp.ru>
   27  */
   28 #ifndef AODVROUTINGPROTOCOL_H
   29 #define AODVROUTINGPROTOCOL_H
   30 
   31 #include "aodv-rtable.h"
   32 #include "aodv-rqueue.h"
   33 #include "aodv-packet.h"
   34 #include "aodv-neighbor.h"
   35 #include "aodv-dpd.h"
   36 #include "ns3/node.h"
   37 #include "ns3/random-variable-stream.h"
   38 #include "ns3/output-stream-wrapper.h"
   39 #include "ns3/ipv4-routing-protocol.h"
   40 #include "ns3/ipv4-interface.h"
   41 #include "ns3/ipv4-l3-protocol.h"
   42 #include <map>
   43 
   44 namespace ns3
   45 {
   46 namespace aodv
   47 {
   53 class RoutingProtocol : public Ipv4RoutingProtocol
   54 {
   55 public:
   56   static TypeId GetTypeId (void);
   57   static const uint32_t AODV_PORT;
   58 
   60   RoutingProtocol ();
   61   virtual ~RoutingProtocol();
   62   virtual void DoDispose ();
   63 
   64   // Inherited from Ipv4RoutingProtocol
   65   Ptr<Ipv4Route> RouteOutput (Ptr<Packet> p, const Ipv4Header &header, Ptr<NetDevice> oif, Socket::SocketErrno &sockerr);
   66   bool RouteInput (Ptr<const Packet> p, const Ipv4Header &header, Ptr<const NetDevice> idev,
   67                    UnicastForwardCallback ucb, MulticastForwardCallback mcb,
   68                    LocalDeliverCallback lcb, ErrorCallback ecb);
   69   virtual void NotifyInterfaceUp (uint32_t interface);
   70   virtual void NotifyInterfaceDown (uint32_t interface);
   71   virtual void NotifyAddAddress (uint32_t interface, Ipv4InterfaceAddress address);
   72   virtual void NotifyRemoveAddress (uint32_t interface, Ipv4InterfaceAddress address);
   73   virtual void SetIpv4 (Ptr<Ipv4> ipv4);
   74   virtual void PrintRoutingTable (Ptr<OutputStreamWrapper> stream) const;
   75 
   76   // Handle protocol parameters
   77   Time GetMaxQueueTime () const { return MaxQueueTime; }
   78   void SetMaxQueueTime (Time t);
   79   uint32_t GetMaxQueueLen () const { return MaxQueueLen; }
   80   void SetMaxQueueLen (uint32_t len);
   81   bool GetDesinationOnlyFlag () const { return DestinationOnly; }
   82   void SetDesinationOnlyFlag (bool f) { DestinationOnly = f; }
   83   bool GetGratuitousReplyFlag () const { return GratuitousReply; }
   84   void SetGratuitousReplyFlag (bool f) { GratuitousReply = f; }
   85   void SetHelloEnable (bool f) { EnableHello = f; }
   86   bool GetHelloEnable () const { return EnableHello; }
   87   void SetBroadcastEnable (bool f) { EnableBroadcast = f; }
   88   bool GetBroadcastEnable () const { return EnableBroadcast; }
   89 
   98   int64_t AssignStreams (int64_t stream);
   99 
  100 protected:
  101   virtual void DoInitialize (void);
  102 private:
  103   
  104   // Protocol parameters.
  105   uint32_t RreqRetries;             
  106   uint16_t RreqRateLimit;           
  107   uint16_t RerrRateLimit;           
  108   Time ActiveRouteTimeout;          
  109   uint32_t NetDiameter;             
  110 
  114   Time NodeTraversalTime;
  115   Time NetTraversalTime;             
  116   Time PathDiscoveryTime;            
  117   Time MyRouteTimeout;               
  118 
  122   Time HelloInterval;
  123   uint32_t AllowedHelloLoss;         
  124 
  128   Time DeletePeriod;
  129   Time NextHopWait;                  
  130   Time BlackListTimeout;             
  131   uint32_t MaxQueueLen;              
  132   Time MaxQueueTime;                 
  133   bool DestinationOnly;              
  134   bool GratuitousReply;              
  135   bool EnableHello;                  
  136   bool EnableBroadcast;              
  137   //\}
  138 
  140   Ptr<Ipv4> m_ipv4;
  142   std::map< Ptr<Socket>, Ipv4InterfaceAddress > m_socketAddresses;
  144   std::map< Ptr<Socket>, Ipv4InterfaceAddress > m_socketSubnetBroadcastAddresses;
  146   Ptr<NetDevice> m_lo; 
  147 
  149   RoutingTable m_routingTable;
  151   RequestQueue m_queue;
  153   uint32_t m_requestId;
  155   uint32_t m_seqNo;
  157   IdCache m_rreqIdCache;
  159   DuplicatePacketDetection m_dpd;
  161   Neighbors m_nb;
  163   uint16_t m_rreqCount;
  165   uint16_t m_rerrCount;
  166 
  167 private:
  169   void Start ();
  171   void DeferredRouteOutput (Ptr<const Packet> p, const Ipv4Header & header, UnicastForwardCallback ucb, ErrorCallback ecb);
  173   bool Forwarding (Ptr<const Packet> p, const Ipv4Header & header, UnicastForwardCallback ucb, ErrorCallback ecb);
  178   void ScheduleRreqRetry (Ipv4Address dst);
  185   bool UpdateRouteLifeTime (Ipv4Address addr, Time lt);
  191   void UpdateRouteToNeighbor (Ipv4Address sender, Ipv4Address receiver);
  193   bool IsMyOwnAddress (Ipv4Address src);
  195   Ptr<Socket> FindSocketWithInterfaceAddress (Ipv4InterfaceAddress iface) const;
  197   Ptr<Socket> FindSubnetBroadcastSocketWithInterfaceAddress (Ipv4InterfaceAddress iface) const;
  199   void ProcessHello (RrepHeader const & rrepHeader, Ipv4Address receiverIfaceAddr);
  201   Ptr<Ipv4Route> LoopbackRoute (const Ipv4Header & header, Ptr<NetDevice> oif) const;
  202 
  204   //\{
  206   void RecvAodv (Ptr<Socket> socket);
  208   void RecvRequest (Ptr<Packet> p, Ipv4Address receiver, Ipv4Address src);
  210   void RecvReply (Ptr<Packet> p, Ipv4Address my,Ipv4Address src);
  212   void RecvReplyAck (Ipv4Address neighbor);
  214   void RecvError (Ptr<Packet> p, Ipv4Address src);
  215   //\}
  216 
  218   //\{
  220   void SendPacketFromQueue (Ipv4Address dst, Ptr<Ipv4Route> route);
  222   void SendHello ();
  224   void SendRequest (Ipv4Address dst);
  226   void SendReply (RreqHeader const & rreqHeader, RoutingTableEntry const & toOrigin);
  232   void SendReplyByIntermediateNode (RoutingTableEntry & toDst, RoutingTableEntry & toOrigin, bool gratRep);
  234   void SendReplyAck (Ipv4Address neighbor);
  236   void SendRerrWhenBreaksLinkToNextHop (Ipv4Address nextHop);
  238   void SendRerrMessage (Ptr<Packet> packet,  std::vector<Ipv4Address> precursors);
  245   void SendRerrWhenNoRouteToForward (Ipv4Address dst, uint32_t dstSeqNo, Ipv4Address origin);
  247 
  248   void SendTo (Ptr<Socket> socket, Ptr<Packet> packet, Ipv4Address destination);
  249 
  251   Timer m_htimer;
  253   void HelloTimerExpire ();
  255   Timer m_rreqRateLimitTimer;
  257   void RreqRateLimitTimerExpire ();
  259   Timer m_rerrRateLimitTimer;
  261   void RerrRateLimitTimerExpire ();
  263   std::map<Ipv4Address, Timer> m_addressReqTimer;
  265   void RouteRequestTimerExpire (Ipv4Address dst);
  267   void AckTimerExpire (Ipv4Address neighbor,  Time blacklistTimeout);
  268 
  270   Ptr<UniformRandomVariable> m_uniformRandomVariable;  
  272   Time m_lastBcastTime;
  273 };
  274 
  275 }
  276 }
  277 #endif /* AODVROUTINGPROTOCOL_H */
