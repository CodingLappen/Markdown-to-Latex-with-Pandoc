
\center\includegraphics[width=\textwidth]{images/p4.png}

# Introduction

## P4 as a Concept

* A program provided by the user.
* Compiler, architecture model and target provided by the manufacturer

\includegraphics[width=\textwidth]{images/architecture.png}

## By P4 supported Concepts
* Header Stack 
  + Parsing multiple types packages
  + Accepting / Rejecting packages while parsing
  + Applying rules

* Pipelining
  + Parser
  + Match-action-pipeline
  + Deparser

* Metadata

# Architecture
## Architecture
\includegraphics[width=\textwidth]{images/pipeline.png}

# Parsing
## Parser 
\center \includegraphics[height=\textheight]{images/parser-states.png}

## Headerstack
**Header stack:**
\center \includegraphics[width=\textwidth]{images/headerstack.png}

## A simple parse graph
\center \includegraphics[height=\textheight]{images/SimpleParser.png}

## Header

\center \includegraphics[width=\textwidth]{images/ethernet.png}

## Header 
~~~ {language=P4 numberstyle=\tiny}
const bit<16> TYPE_IPV4 = 0x800;

typedef bit<9>  egressSpec_t;
typedef bit<48> macAddr_t;
typedef bit<32> ip4Addr_t;
typedef bit<128> ip6Addr_t;

header ethernet_t {
    macAddr_t dstAddr;
    macAddr_t srcAddr;
    bit<16>   etherType;
}

struct headers {
  ethernet_t ethernet;
  ipv4_t ipv4;
}
~~~ 


## Example {.allowframebreaks}
* \texttt{Packet\_in} is important
~~~ {language=P4 numberstyle=\tiny}
  parser Parser(packet_in packet,
                  out headers hdr,
                  inout metadata meta,
                  inout standard_metadata_t standard_metadata) {
  
      state start {
          transition parse_ethernet;
      }
  
      state parse_ethernet {
          packet.extract(hdr.ethernet);
          transition select(hdr.ethernet.etherType) {
              TYPE_IPV4: parse_ipv4;
              default: accept;
          }
      }
  
      state parse_ipv4 {
          packet.extract(hdr.ipv4);
          transition accept;
      }
  }
~~~


# Match+Action pipeline 
1. Table has a set of actions 
2. Table has a key set for lookup tables.
3. Actions have to be inserted
4. There are three kinds of key matching methods (Lookup methods):
  * exact
  * ternary
  * lpm

## Actions
~~~ {language=P4 numberstyle=\tiny}
action a(inout header_t, inout metadata_t) {

}
action b (inout metadata_t,bit<4> type) {
  /* Manipulate metadata */
}
~~~

## Table

~~~ {language=P4 numberstyle=\tiny frame=single}
table ipv4_lpm {
        key = { hdr.ipv4.dstAddr: lpm; }
        actions = {
            ipv4_forward;
            drop;
            NoAction;
        }
        size = 1024;
        default_action = drop();
    }
~~~ 

## Control
control = actions + tables + application_logic

actions = actions +action | $\epsilon$

tables = tables + table | $\epsilon$

~~~ {language=P4 numberstyle=\tiny frame=single}
control IngressExample(inout headers hdr, inout metadata meta, inout standard_metadata_t standard_metadata) {
apply {
  if (hdr.ipv6.isValid() && !hdr.ipv4.isValid()){
    ipv6_lpm.apply()
  }
  if (hdr.ipv4.isValid() && !hdr.ipv6.isValid()){
    ipv4_lpm.apply()
  }
}


}
~~~

## Primitive Actions {.allowframebreaks}
  + \texttt{add\_header}
  + \texttt{copy\_header}
  + \texttt{remove\_header}
  + \texttt{modify\_field}
  + \texttt{add\_to\_field}
  + \texttt{add}
  + \texttt{substract\_from\_field}
  + \texttt{substract}
  + \texttt{modify\_field\_with\_with\_hash\_based\_offset}
  + \texttt{modify\_field\_rng\_uniform}
  + \texttt{bit\_and}
  + \texttt{bit\_or}
  + \texttt{bit\_xor}
  + \texttt{shift\_left}
  + \texttt{shift\_right}
  + \texttt{truncate}
  + \texttt{drop}
  + \texttt{no\_op}
  + \texttt{pop}
  + \texttt{count}
  + \texttt{execute\_meter}
  + \texttt{register\_read}
  + \texttt{register\_write}
  + \texttt{generate\_digest}
  + \texttt{resubmit}
  + \texttt{recirculate}
  + \texttt{clone\_ingress\_pkt\_to\_ingress}
  + \texttt{clone\_egress\_pkt\_to\_ingress}
  + \texttt{clone\_ingress\_pkt\_to\_egress}
  + \texttt{clone\_egress\_pkt\_to\_egress}

# Deparsing 
## Emiting header to corresponding header stack
  * \texttt{Packet\_out} parameter
  * \texttt{headers} struct as incoming and maybe \texttt{metadata\_t} as incoming parameter
~~~ {language=P4 numberstyle=\tiny}
control Deparser(packet_out packet, in headers hdr) {
  apply {
      packet.emit(hdr.ethernet);
    if (hdr.ipv6.isValid() && !hdr.ipv4.isValid()){
        packet.emit(hdr.ipv4);
    }
    if (hdr.ipv4.isValid() && !hdr.ipv6.isValid()){
        packet.emit(hdr.ipv6);
  }
}
~~~

# Defining the pipeline

**As a reminder**:

\includegraphics[width=\textwidth]{images/pipeline.png}

## Example

* There can be more than two controls
~~~ {language=P4 numberstyle=\tiny}

V1Switch(
  Parser(),
  Ingress(),
  Egress(),
  Deparser()
) main;
~~~

# Usage
* Load balancing (\underline{\texttt{count}} action for example)

* Package mirroring / Network debugging \linebreak
  (\texttt{metadata}, \texttt{clone}   vs. \texttt{ttl}, \texttt{ping}, \texttt{traceroute})

* Dynamic network creation / easy implementation of new protocols (time sensitive networking (TSN) for example)

* Teaching people basic concepts about routing

# Sources
## References
* **P4 Homepage**:
  * [\color{blue}{\underline{P4}}](https://p4.org/p4-spec)

* **P4 Specification on Github** (Images):
  * [\color{blue}{\underline{p4-spec}}](https://github.com/p4lang/p4-spec)

* **P4 Tutorial**:
  * [\color{blue}{\underline{P4 Tutorial}}](https://github.com/p4lang/tutorials/)

* **Introduction Paper**:
  * [\color{blue}{\underline{P4: Programming Protocol-Independent Packet Processors}}](https://www.sigcomm.org/sites/default/files/ccr/papers/2014/July/0000000-0000004.pdf)
