> 1.  **Architecture and Explanation**:
> 
> -   Could you provide a detailed explanation of the architecture of Builder Vault TSM  [https://builder-vault-tsm.docs.blockdaemon.com/docs/what-is-builder-vault-tsm#5-tsm-web3-wallet-use-case](https://builder-vault-tsm.docs.blockdaemon.com/docs/what-is-builder-vault-tsm#5-tsm-web3-wallet-use-case "https://builder-vault-tsm.docs.blockdaemon.com/docs/what-is-builder-vault-tsm#5-tsm-web3-wallet-use-case")?

-   The application architecture is best depicted here: [https://builder-vault-tsm.docs.blockdaemon.com/docs/what-is-builder-vault-tsm#3-builder-vault-tsm-architecture](https://builder-vault-tsm.docs.blockdaemon.com/docs/what-is-builder-vault-tsm#3-builder-vault-tsm-architecture "https://builder-vault-tsm.docs.blockdaemon.com/docs/what-is-builder-vault-tsm#3-builder-vault-tsm-architecture")
-   A single Builder Vault TSM comprising of multiple TSM nodes can be used to generate multiple keys for performing cryptographic operations. At no point does the full private key exist in any one place.
-   The TSM can be deployed in a dynamic fashion, allowing for nodes to be added or removed from the cluster, without disrupting operations.
-   The solution follows a shared-nothing architecture, where nodes are independent from one another, and can be deployed, scaled, administered and operated separately.
-   The TSM architecture is flexible in its support for a multitude of mixed environments from public cloud, to on-premise, to mobile devices.
-   TSM nodes perform many rounds of communication when performing multi-party computations. This inter-node TLS communication can be either point-to-point over protocols such as TCP and websockets, or indirect through a message broker with AMQP, for more complex networking setups.
-   The node-to-node communication is either end-to-end encrypted using TLS. Or, when using a message broker, there is TLS between the node and the broker, with another encryption layer on top of that, so the message broker is not trusted in any way.
-   Sensitive application state is encryption at the application level prior to being persisted to each node's local database.
-   Client SDKs are used to administer and operate the TSM.
-   Client SDK authentication is handled by each TSM node with support for common protocols such as OIDC, client certificates, and token based auth.

> -   What are the components of Builder Vault TSM, and what are their functions?

-   The Builder Vault solution comprises the following:

-   client SDK instances across numerous languages for TSM administration and operations.
-   TSM node instances perform functions across numerous supported MPC protocols, such as quorum approvals, managing key shares, and cryptographic operations
-   message broker (optional) - used if scenarios where point-to-point is not suitable
-   embedded node (optional) - used with non-standard devices where long running network connectivity is not suitable
-   audit server (optional) - used to receive audit events from the TSM

> -   Can you provide a diagram or visual representation of the implementation architecture?

-   The attached PDF has examples of a Builder Vault Core AWS implementation, and a Builder Vault Core with dynamic third node implementation. On the first page two TSM instances are deployed for basic use. AWS Nitro enclave confidential computing is used to protect the TSMs at runtime, however confidential computing does not protect against insider threats. If required, a third TSM can be joined dynamically from a separate AWS organization that is managed by a separate group of administrators (see the second page for this).

> 2.  **Necessary Use Cases for Embedded Node**:
> 
> -   In which specific use cases is it necessary to use an embedded node with Builder Vault TSM?

-   MPC nodes are distributed as container images. Some environments, in particular mobile devices, are not well suited for running containers. So for running MPC nodes in these environments (iOS / Android), a library that contains both the SDK and the MPC node itself (embedded in the library), is available. The main use case for embedded MPC nodes is running the MPC node on mobile devices.

> -   What are the advantages and disadvantages of using an embedded node in these scenarios?

-   Advantages include wider device compatibility and operability through inconsistent networking
-   Disadvantages include additional overheads in wrapping the embedded node within a device compatible parent application, and adjusting for device initiated MPC workflows.

> -   Can you provide examples of real-world applications where an embedded node would be beneficial?

-   Should a service provider wish to offer self-custodial web3 services to customers, a key share could be generated and held by the customer on their smartphone. Similarly, multiple customers can share keys from the same wallet. Some examples scenarios below:

> 2.  **2/3 Model Explanation**:
> 
> -   Why is it required for all nodes to perform a partialSign in the 2/3 multi-signature model, instead of just two nodes being sufficient?

-   With some protocols, such as SEPH18S, cryptographic operations may have a different threshold to the key recovery. For example with SEPH18S t+1 shares are required to recover the key while 2t+1 shares are required for signing. With DKLs19 both operations and key construction require t+1.

> -   What are the security implications of this requirement, and how does it contribute to the overall security of the system?

-   Different protocols have different trade offs, not only around security but also for example speed or availability. Thomas has provided intuitive terminology for these: the thresholds as follows:

-   Security threshold (t) - is the number of corrupted nodes we can tolerate in order to keep the key secret
-   Operational threshold - is the number of corrupted/crashed nodes we can tolerate and still run the actual signing protocol

> 2.  **Terminology Clarification**:
> 
> -   What is a "player" in the context of Builder Vault TSM?

-   The "player" represents an individual participant within a party performing multi-party computations.

> -   What is the purpose of the "createInitial" function, and how is it used in the key generation process?

-   With version 1 of the SDK, a new deployment of a TSM node requires that it be initialized with both a user and an administrator account. The "createInitialAdmin" method is used to perform this generate of credentials. This is an unauthenticated call but can only be performed once. Note, the feature is being deprecated in version 2 of the SDK, in favor of API keys or mTLS client authentication certificates, both of which must be defined in the configuration.

> -   What is the difference between an "admin" and a "user" in Builder Vault TSM?

-   The admin account can only perform functions such as managing users and quiescing the system for backups. It cannot perform key operations.
-   User accounts cannot perform admin operations and can only perform key management operations, such as key generation, signing, hashing, etc. And again, admin/user are deprecated in SDKv2, in favor of using API keys.

> -   What is a "session id" and how does it relate to the key generation and signing processes?

-   Unlike with the demo setup examples, it is best practice that each individual TSM instance be operated by its own SDK instance, as opposed to a single instance of the SDK managing all TSM instances. In these scenarios, one instance of the SDK needs to generate a session ID (a random string) and share it with the other separate SDK instances, in order to ensure coordination of the operation across all TSM nodes.

> -   What is a "session num" and how is it used in the key generation and signing processes?

-   I'm not aware of a session num parameter in the SDK. Could this be a duplicate of session id?

> -   What is a "key id" and how is it used in the key generation and signing processes?

-   The key ID is a unique identifier which is associated with each key share in each respective TSM node. From the SDK point of view the Key ID represents the logical grouping of all key shares across all TSM nodes.

> -   What is the purpose of the "keygenWithSessionID" function, and how does it fit into the key generation process?

-   In the scenarios where each TSM node is operated by its own instance of the SDK, the key generation method can be supplied with the shared session ID to facilitate coordination of the operation over multiple nodes within the timeout period.

> -   What is a "presignature" and how is it used in the signing process?

-   Presignatures can be generated interactively between TSM nodes, ahead of signing operations. Presignatures speed up signing operations by reducing the number of MPC communication rounds between players.

> -   What is a "presigID" and how does it relate to the presignature process?

-   Each presignature is generated from the key ID ahead of signing operations. When signing commences the presignatures need to be uniquely identified and referenced. presignatureIDs is the variable name used in some of the code examples for this.

>
