# IOPPY

Disclamer: no realy meaning for a name.

This description is a result of my thoughts on hybrid practical systems that use non-O(1) proof-systems at what could be called "layer 1" and later use a O(1) proof systems on "layer 2" to make a succinct proofs of a statement that "For a given Layer 1 problem that has a corresponding verifier encoded into the statement of Layer 2 there was provided a valid proof that satisfied a verifier". In a nutshell - write a SNARK that internally verifies a STARK: all the IOP witness can be discarded as a private input to the SNARK, and STARK verifier equation is encoded as R1CS statement.

This construction was also discussed with Ariel Gabizon, Daira Hopwood, Izaak Meckler and other during ZCon 1 in Split, June 2019.

## Overview

By the mid of 2019 there exist a large number of various proof systems that have a public coin (transparent, trustless) or private coin setup (private coin setup == trusted setup). Main advantage of proof system with a trusted setup is succinctness in proof size and verifier's work. Such systems are usually require pairing friendly curves and have interesting instantiations like recursive constructions (recursive SNARKS) or embedded curves (e.g. JubJub construction by ZCash or instance from Zexe). Recursive constructions are the most interesting that up to some simplification can aggregate proofs for very large set of statements.

One problem with recursive cycles in that curves with 128 bits of security require operations in ~700 bit fields and ~700 bit groups. Also a search for such curves is non-trivial, taking into account additional requirements like high 2-adicity or in general smooth multiplicative group structure.

SHARKs construction was proposed by ZCash for purposes on aggregation of non-O(1) transparent proofs (w/o private coin setup). This approach allows to find if private coin setup was compromised. Ideal instantiation of such construction taking into account recursion would require one to find a set of curves like `BLS12 -> ? -> MNT4/6` or at least a pair `Curve -> MNT4/6` where `->` symbol means that base field of one curve is equal to the main group order of another curve to allow embedding. Unfortunately search for such combination of curves can be too difficult.

## Proposed solution and concrete example

There exist public coin proof systems that have verifier's work sublinear in "instance size" of the statement (circuit). Example of such proof system in Stark that requires verifier to only check polynomial constraints for a set of points that are provided by prover through the oracle (interactive oracle proof, IOP). Stark has O(polylog(n)) communication complexity that can be a problem if verification is done using e.g. a smart-contract in a public blockchain. In terms of requirements to instanciate Stark has only a prime field (here only Starks over prime fields are discussed) with large 2-adicity. Verifier's operations consist of verification of Merkle path's for elements provided by oracle and some arithmetic in a prime field for constraints verification and FRI protocol.

As it happens any pairing friendly curve that is SNARK-friendly (high 2-adicity of the main group order for FFT) leads to possibility to instanciate an arithmetic circuit over a field with modulus equal to the main order group. It naturally leads to posibility to instantiate STARK over exactly the same field. If one assumes that IOP itself is instanciated using one of the "SNARK-friendly" hashes (arithmetic hashes like Rescue or Poseidon, may be even more standard Pedersen hashes) then a circuit to implement a STARK verifier can be quite compact. There are basically two operations that verifier has to do:
- Check openings of oracle, that is essencially hashing of a size ~ O(log(T*W)) where T is a length of the trace of AIR arithmetization for STARK and W is a number of "registers" in such arithmetization
- Verify polynomial constraints, that are degree bounded by the STARK protocol and are usually made low-degree
- Perform FRI protocol operations that involve some multiplications/squarings in a field and linear interpolation 
  
Such a simple structure of the verifier allows one to expect low complexity of a corresponding circuit in assumption of a proper hash function for oracle proofs. 

## Problems

Absence of an open and well-documented STARK prover does not allow immediate testing of such approach. Also, design of AIR arithmetization of the problem is much more challenging than design of R1CS arithemtization.

## Hybridization with other proof systems

In such approach one is largely limited by use of SNARKs as a "Layer 2" system to have all the benefits of succinctness and recursive cycles. 

As for a Layer 1 one could think about use of Aurora proof system, that also requires only field operations and IOP verification by the verifier. Unfortunately, Aurora requires verifier to evaluate a polynomial of the degree equal to the number of variables in R1CS instance, that would lead to the circuit of Layer 2 statement to be in order of size of Layer 1 statement.

If at some point other proof system (may be IOP based system) like Aurora is introduced for non-uniform circuits with, for example, verifier's work being logarithmic in the size of the arithmetization of original statement, such proof system could be immediately used for construction of a hybrid approach. But it also would lead to self-recursiveness in asumption of a proper hash function for IOP.

## Authors

Alex Vlasov, [@shamatar](https://github.com/shamatar),  alex.m.vlasov@gmail.com, Matter Labs