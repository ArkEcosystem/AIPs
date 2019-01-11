<pre>
  AIP: 1
  Title: AIP Purpose and Guidelines
  Author: Kristjan Kosic <kristjan@ark.io> - based on BIPS/BIP-0001
  Discussions-To: https://github.com/arkecosystem/AIPS/issues
  Status: Active
  Type: Process
  Created: 2019-01-10
</pre>

# What is a AIP?

AIP stands for Ark Improvement Proposal. A AIP is a design document providing information to the Ark community, or describing a new feature for Ark or its processes or environment. The AIP should provide a concise technical specification of the feature and a rationale for the feature.

We intend AIPs to be the primary mechanisms for proposing new features, for collecting community input on an issue, and for documenting the design decisions that have gone into Ark. The AIP author is responsible for building consensus within the community and documenting dissenting opinions.

Because the AIPs are maintained as text files in a versioned repository, their revision history is the historical record of the feature proposal.

# AIP Types

There are three kinds of AIP:

* A Standards Track AIP describes any change that affects mostly Ark core implementations, such as a change to the network protocol, a change in block or transaction validity rules, or any change or addition that affects the interoperability of applications.
* An Informational AIP describes a Ark's design issue, or provides general guidelines or information to the community, but does not propose a new feature. Informational AIPs do not necessarily represent a community consensus or recommendation, so users and implementors are free to ignore Informational AIPs or follow their advice.
* A Process AIP describes a process surrounding Ark, or proposes a change to (or an event in) a process. Process AIPs are like Standards Track AIPs but apply to areas other than the Ark protocol itself. They may propose an implementation, but not to Ark's codebase; they often require community consensus; unlike Informational AIPs, they are more than recommendations, and users are typically not free to ignore them. Examples include procedures, guidelines, changes to the decision-making process, and changes to the tools or environment used in Ark development. Any meta-AIP is also considered a Process AIP.

# AIP Work Flow

The AIP process begins with a new idea for Ark. Each potential Ark must have a champion - someone who writes the AIP using the style and format described below, shepherds the discussions in the appropriate forums, and attempts to build community consensus around the idea. The AIP champion (a.k.a. Author) should first attempt to ascertain whether the idea is AIP-able. Sharing the idea or discussion on our slack channell is the best way to go about this.

Vetting an idea publicly before going as far as writing a AIP is meant to save both the potential author and the wider community time. Many ideas have been brought forward for changing Ark that have been rejected for various reasons. Asking the community first if an idea is original helps prevent too much time being spent on something that is guaranteed to be rejected based on prior discussions (searching the internet does not always do the trick). It also helps to make sure the idea is applicable to the entire community and not just the author. Just because an idea sounds good to the author does not mean it will work for most people in most areas where Ark is used. Small enhancements or patches often don't need standardisation between multiple projects; these don't need a AIP and should be injected into the relevant development work flow with a patch submission to the our github page in the form of a Pull Request. 

AIP authors are responsible for collecting community feedback on both the initial idea and the AIP before submitting it for review. However, wherever possible, long open-ended discussions on public mailing lists should be avoided. Strategies to keep the discussions efficient include: openning up a separate AIP discussion issue for the topic. 

It is highly recommended that a single AIP contain a single key proposal or new idea. The more focused the AIP, the more successful it tends to be. If in doubt, split your AIP into several well-focused ones.

The Ark Team editors assign AIP numbers and change their status. The AIP editor reserves the right to reject AIP proposals if they appear too unfocused or too broad.

Authors MUST NOT self assign AIP numbers, but should use an alias such as "AIP-champion-consensus-improvements" which includes the author's name/nick and the AIP subject.

If the AIP editor approves, he will assign the AIP a number, label it as Standards Track, Informational, or Process, give it status "Draft". The AIP editor will not unreasonably deny a AIP. Reasons for denying AIP status include duplication of effort, disregard for formatting rules, being too unfocused  or too broad, being technically unsound, not providing proper motivation or addressing backwards compatibility. For a AIP to be accepted it must meet certain minimum criteria. It must be a clear and complete description of the proposed enhancement. The enhancement must represent a net improvement. The proposed implementation, if applicable, must be solid and must not complicate the protocol unduly.

The AIP author may update the Draft as necessary in the git repository. Updates should be submitted by the author as pull requests.

Standards Track AIPs consist of two parts, a design document and a reference implementation. The AIP should be reviewed and accepted before a reference implementation is begun, unless a reference implementation will aid people in studying the AIP. Standards Track AIPs must include an implementation - in the form of code, a patch, or a URL to same - before it can be considered Final.

Once a AIP has been accepted, the reference implementation must be completed. When the reference implementation is complete and accepted the status will be changed to "Final".

An AIP can also be assigned status "Deferred". The AIP author or editor can assign the AIP this status when no progress is being made on the AIP. Once a AIP is deferred, the AIP editor can re-assign it to draft status.

A AIP can also be "Rejected". Perhaps after all is said and done it was not a good idea. It is still important to have a record of this fact.

AIPs can also be superseded by a different AIP, rendering the original obsolete. This is intended for Informational AIPs, where version 2 of an API can replace version 1.

The possible paths of the status of AIPs are as follows:

<img src=../assets/aip-1/process-01.png></img>

Some Informational and Process AIPs may also have a status of "Active" if they are never meant to be completed. E.g. AIP 1 (this AIP).

# What belongs in a successful AIP?

Each AIP should have the following parts:

* Preamble - RFC 822 style headers containing meta-data about the AIP, including the AIP number, a short descriptive title (limited to a maximum of 44 characters), the names, and optionally the contact info for each author, etc.

* Abstract - a short (~200 word) description of the technical issue being addressed.

* Copyright/public domain - Each AIP must be licensed under the MIT License.

* Specification - The technical specification should describe the syntax and semantics of any new feature. The specification should be detailed enough to allow competing, interoperable implementations.

* Motivation - The motivation is critical for AIPs that want to change the Ark protocol. It should clearly explain why the existing protocol specification is inadequate to address the problem that the AIP solves. AIP submissions without sufficient motivation may be rejected outright.

* Rationale - The rationale fleshes out the specification by describing what motivated the design and why particular design decisions were made. It should describe alternate designs that were considered and related work, e.g. how the feature is supported in other languages.

* The rationale should provide evidence of consensus within the community and discuss important objections or concerns raised during discussion.

* Backwards Compatibility - All AIPs that introduce backwards incompatibilities must include a section describing these incompatibilities and their severity. The AIP must explain how the author proposes to deal with these incompatibilities. AIP submissions without a sufficient backwards compatibility treatise may be rejected outright.

* Reference Implementation - The reference implementation must be completed before any AIP is given status "Final", but it need not be completed before the AIP is accepted. It is better to finish the specification and rationale first and reach consensus on it before writing code.

* The final implementation must include test code and documentation appropriate for the Ark protocol.

# AIP Formats and Templates

AIPs should be written in markdown format.

## AIP Header Preamble

Each AIP must begin with an RFC 822 style header preamble. The headers must appear in the following order. Headers marked with "*" are optional and are described below. All other headers are required.

<pre>
  AIP: <AIP number>
  Title: <AIP title>
  Author: <list of authors' real names and optionally, email addrs>
* Discussions-To: <AIP issue link>
  Status: <Draft | Active | Accepted | Deferred | Rejected |
           Withdrawn | Final | Superseded>
  Type: <Standards Track | Informational | Process>
  Created: <date created on, in ISO 8601 (yyyy-mm-dd) format>
* Replaces: <AIP number>
* Superseded-By: <AIP number>
* Resolution: <url>
</pre>

The Author header lists the names, and optionally the email addresses of all the authors/owners of the AIP. The format of the Author header value must be

`Random J. User <address@dom.ain>`

if the email address is included, and just

`Random J. User`

if the address is not given.

If there are multiple authors, each should be on a separate line following RFC 2822 continuation line conventions.

Note: The Resolution header is required for Standards Track AIPs only. It contains a URL that should point to an email message or other web resource where the pronouncement about the AIP is made.

While a AIP is in discussions (usually during the initial Draft phase), a Discussions-To header will indicate the URL where the AIP is being discussed. No Discussions-To header is necessary if the AIP is being discussed privately with the author.

The Type header specifies the type of AIP: Standards Track, Informational, or Process.

The Created header records the date that the AIP was assigned a number.

AIPs may have a Requires header, indicating the AIP numbers that this AIP depends on.

AIPs may also have a Superseded-By header indicating that a AIP has been rendered obsolete by a later document; the value is the number of the AIP that replaces the current document. The newer AIP must have a Replaces header containing the number of the AIP that it rendered obsolete.

## Auxiliary Files

AIPs may include auxiliary files such as diagrams. Image files should be included in a subdirectory for that AIP in the assets folder. Auxiliary files must be named aip-XXXX-Y.ext, where "XXXX" is the AIP number, "Y" is a serial number (starting at 1), and "ext" is replaced by the actual file extension (e.g. "png").

## Transferring AIP Ownership

It occasionally becomes necessary to transfer ownership of AIPs to a new champion. In general, we'd like to retain the original author as a co-author of the transferred AIP, but that's really up to the original author. A good reason to transfer ownership is because the original author no longer has the time or interest in updating it or following through with the AIP process, or has fallen off the face of the 'net (i.e. is unreachable or not responding to email). A bad reason to transfer ownership is because you don't agree with the direction of the AIP. We try to build consensus around a AIP, but if that's not possible, you can always submit a competing AIP.

If you are interested in assuming ownership of a AIP, send a message asking to take over, addressed to both the original author and the AIP editor. If the original author doesn't respond to email in a timely manner, the AIP editor will make a unilateral decision (it's not like such decisions can't be reversed :).

## AIP Editor Responsibilities & Workflow

The AIP editor subscribes to the Ark's AIPs github notifications. For each new AIP that comes in an editor does the following:

* Read the AIP to check if it is ready: sound and complete. The ideas must make technical sense, even if they don't seem likely to be accepted.
* The title should accurately describe the content.
* The AIP draft must have open discussion to the AIPs github as an open issue with correct naming.
* Motivation and backward compatibility (when applicable) must be addressed.
* The defined Layer header must be correctly assigned for the given specification.
* Licensing terms must be acceptable for AIPs.

If the AIP isn't ready, the editor will send it back to the author for revision, with specific instructions.

Once the AIP is ready for the repository it should be submitted as a "pull request" to the AIPs git repository where it may get further feedback.

The AIP editor will:

* Assign a AIP number in the pull request.
* Merge the pull request when it is ready.
* List the AIP in AIP list.

The AIP editors are intended to fulfill administrative and editorial responsibilities. The AIP editors monitor AIP changes, and update AIP headers as appropriate.

# History

This document was derived heavily from Bitcoin's BIP-0001 and BIP-0002. In many places text was simply copied and modified. Previous authors are not responsible for its use in the Ark Improvement Process, and should not be bothered with technical questions specific to Ark or the AIP process. Please direct all comments to the AIP editors or the Ark official slack.

# Changelog

11 Jan 2019 - Added initial version, forked from BIP-0001 BIPS proposals.

