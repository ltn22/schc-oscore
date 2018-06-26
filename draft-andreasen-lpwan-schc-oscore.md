---
stand_alone: true
ipr: trust200902
docname: draft-andreasen-lpwan-schc-oscore-00
cat: info
pi:
  symrefs: 'yes'
  sortrefs: 'yes'
  strict: 'yes'
  compact: 'yes'
  toc: 'yes'
title: Compression of OSCORE with SCHC
abbrev: SCHC OSCORE
wg: lpwan Working Group
author:
- ins: R. Andreasen
  name: Ricardo Andreasen
  org: IMT Atlantique
  street:
  - 2 rue de la Chataigneraie
  - CS 17607
  city: 35576 Cesson-Sevigne Cedex
  country: France
  email: ricardo.andreasen@imt-atlantique.net
- ins: L. Toutain
  name: Laurent Toutain
  org: IMT-Atlantique
  street:
  - 2 rue de la Chataigneraie
  - CS 17607
  city: 35576 Cesson-Sevigne Cedex
  country: France
  email: Laurent.Toutain@imt-atlantique.fr
normative:
  I-D.ietf-core-object-security:
  I-D.ietf-lpwan-coap-static-context-hc:
informative:  

--- abstract

This document defines the SCHC compression for OSCORE

--- middle

# Introduction {#Introduction}

OSCORE {{I-D.ietf-core-object-security}}  defines a end-to-end protect for CoAP messages. This
document defines how CoAP compression rules {{I-D.ietf-lpwan-coap-static-context-hc}} can be 
apply to OSCORE. 

# OSCORE 

For a more detailed explanation of OSCORE please refer to {{I-D.ietf-core-object-security}}. This 
chapter explain briefly how is formed a OSCORE message from a regular CoAP message.

OSCORE split a CoAP message into two headers. The outer header will carry information that can be read by any cache of proxy. This header follows the structure defined for a CoAP message. Intermediary equipments such as proxy and cache may process this information. 
The inner header is viewed as the payload of the outer header and is encrypted by the sender. The format is slightly different form a CoAP message. 

OSCORE classify CoAP option from the original message in three category:

 * E for Encrypted options, these options will be stored in the inner header and therefore encrypted for transmission.
   
 * I for Integrity protected options, these options are stored in the outer header and cab be read by any intermediary equipment. They cannot be modified during the transit since their integrity is protected.
 
 * U for unprotected options, these options are stored in the outer header and can be modified during transit.
 
 OSCORE will add to the outer header on object security option. This option informs that OSCORE is used and may carry some informations to help the other end to decrypt the message.
 
 
~~~~
 
                      Orignal CoAP Message
                   +-+-+---+-------+---------------+
                   |v|t|tkl| code  |  Msg Id.      |
                   +-+-+---+-------+---------------+....+
                   | Token                              |
                   +-------------------------------.....+
                   | Options (IEU)            |
                   .                          .
                   .                          .
                   +------+-------------------+ 
                   | 0xFF |
                   +------+------------------------+
                   |                               |
                   |     Payload                   |
                   |                               |
                   +-------------------------------+      
                          /                \ 
                         /                  \
                        /                    \
                       /                      \
     Outer Header     v                        v  Inner header
  +-+-+---+--------+---------------+          +-------+
  |v|t|tkl|new code|  Msg Id.      |          | code  |
  +-+-+---+--------+---------------+....+     +-------+-----......+
  | Token                               |     | Options (E)       |
  +--------------------------------.....+     +-------+------.....+
  | Options (IU)             |                | OxFF  |
  .                          .                +-------+-----------+
  . OSCORE Option            .                |                   |
  +------+-------------------+                | Payload           |
  | 0xFF |                                    |                   |
  +------+                                    +-------------------+

~~~~
{: #Fig-inner-outer title='OSCORE inner and outer header form a CoAP message'}  

{{Fig-inner-outer}} show how the division into inner and outer header is done.
The code of the original message is hidden and a default value (POST or FETCH) replaces it
in the outer header. The original code is stored in first position of the inner header.

The inner header is then encrypted, some additional authenticated data (including I options
and an initialization vector) 
are added into the process.

The result is an encrypted text.

The OSCORE message is composed of the outer options followed by date, composed of an
partial IV and the encrypted message.

~~~~
 
   
     Outer Header                           
  +-+-+---+--------+---------------+          
  |v|t|tkl|new code|  Msg Id.      |          
  +-+-+---+--------+---------------+....+     
  | Token                               |     
  +--------------------------------.....+     
  | Options (IU)             |               
  .                          .               
  . OSCORE Option            .               
  +------+-------------------+               
  | 0xFF |                                  
  +------+-------------------------+
  |                                |
  |  Encrypted Inner Header and    |
  |  Payload                       |
  |                                |
  +--------------------------------+
   
~~~~
{: #Fig-full-oscore title='OSCORE message'}  

{{Fig-full-oscore}} describes the OSCORE message format. 

~~~~

          0 1 2 3 4 5 6 7 <--------- n bytes ------------->
         +-+-+-+-+-+-+-+-+---------------------------------
         |0 0 0|h|k|  n  |      Partial IV (if any) ...
         +-+-+-+-+-+-+-+-+---------------------------------

          <- 1 byte -> <------ s bytes ----->
         +------------+----------------------+------------------+
         | s (if any) | kid context (if any) | kid (if any) ... |
         +------------+----------------------+------------------+

~~~~
{: #Fig-object-sec-opt title='Object Security Option'}  

{{Fig-object-sec-opt}}, taken from {{I-D.ietf-core-object-security}}, gives the format 
of the OSCORE option.

The first 3 bits are set to 0. The next bit h when set on 1 tells that a key identifier
context int the OSCORE option, the k indicates that a kid is also present. 

The 3 n bits give the size of the partial IV stored in the OSCORE option. 

if h bit is set, then a length and the kid context value is stored after the PIV and
finally the kid can be sent.


# SCHC for OSCORE

## inner header compression

## outer header compression

# Performance analysis

# Acknowledgements

Thanks to 

--- back

