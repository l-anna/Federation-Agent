###########################################################################
#Copyright 2016 Anna Levin, Liran Schour - IBM
#
#   Licensed under the Apache License, Version 2.0 (the "License");
#   you may not use this file except in compliance with the License.
#   You may obtain a copy of the License at
#
#       http://www.apache.org/licenses/LICENSE-2.0
#
#   Unless required by applicable law or agreed to in writing, software
#   distributed under the License is distributed on an "AS IS" BASIS,
#   WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
#   See the License for the specific language governing permissions and
#   limitations under the License.
############################################################################
What's Net-fa
=============
Net-fa is a Ryu based application that implements federation agent OF
controller.

Net-fa is a generic SDN federation agent that exposes REST management and 
control API to the Cloud Management Software and between network federation 
agents peers.
In order to support a specific SDN implementation, one should write its own 
FaController class that is derived from the abstract FaSdnController class.
Minimal implementation (L2 ARP based control) should implement the following 
methods: 
    register_vnid() and get_module_name().

Basic architecture:
-------------------

                             +------------------------+
                             |                        |
       RYU infratstructure   |    RYU OF controller   |
                             |                        |
                             +------------------------+
               ------------------|------------------|---------------------
                                 |            +--------------+                 
                        +-----------------+   |              |
       RYU application  | instance derived|<->|    NetFA     |
                        |    from         |   |              |
                        | FaSdnController |   +--------------+
                        +-----------------+          |
                               |                     |
                          +---------+           +---------+  Tunnels to peers
                          | Network |+          | FA DP   |<----------------->
                          +---------+|          +---------+
                           +-----|---+            | | | |
                                 +----------------+ 
                           (patch port per registered network)

Net-fa comes with a specific reference SDN implementation for OVN in 
fa_ovn_controller.py (FaOvnController).
If you want Net-fa to work with your own SDN implementation, take a look
in FaOvnController as a reference implementation and write a class that is
dereived from the abstract class FaSdnController. 

Net-FA OVN implementatiopn was tested on a Fedora21 but should work on any
Linux distribution with kernel>=3.18 (Geneve support)

Quick Start
===========
Install the following packages: python-pip, python-dev python-neutronclient python-keystoneclient
Install Openvswitch from source

% git clone <net-fa Git repository>
% git checkout netfa
% cd net-fa
% sudo pip install -r requirments.txt
% sudo python ./setup.py install
# OR: % cd net-fa; python ./setup.py develop 

# To setup FA ovs datapath:
  Edit OVS_DIR
% sudo setup-fa-ovs.sh <host_ip>

# If you run over OVN you will need to run setup-ovn.sh:
% sudo setup-ovn.sh <host_ip> <ovn_db_ip>

Copy and edit netfa.conf file:
% cp netfa.conf.sample netfa.conf
( Edit netfa.conf...)
udo setup-fa-ovs.sh
% sudo ryu-manager --verbose --wsapi-port=4567 --ofp-tcp-listen-port=1234 --config-file=./netfa.conf ./netfa/net_fa.py <file contains a class derived from FaSdnController e.g ./netfa/fa_ovn_controller.py>

Ryu-manager runs 2 Ryu applications net_fa.py that implements all the REST 
APIs and controlling the FA datapath and a class that implements 
FaSdnController to communicate with teh specific SDN implementation 
(e.g FaOvnController)

We have a script for sharing networks of 2 OpenStack clouds according to the 
netfa.mgmt file. The script generate Net-FA REST API to share tenant's 
networks.

% python ./fc-mgmt-share.py
