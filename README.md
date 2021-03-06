[![Build Status](https://travis-ci.org/ga4gh/cgtd.svg?branch=master)](https://travis-ci.org/ga4gh/cgtd)

# The Cancer Gene Trust
[The Cancer Gene Trust](http://www.cancergenetrust.org) is a simple, real-time, global network for sharing somatic cancer data and associated clinical information. It's architecture is a decentralized, distributed content addressable real-time database. Stewards working directly with patient-participants publish de-identified genomic and basic clinical data. Researchers who find rare variants or combinations of variants in this global resource that are associated with specific clinical features of interest may then contact the data stewards for those participants. A submission consists of a manifest containing fields and references to files by multihash. Initial submissions will likely include de-identified clinical data, a list of somatic mutations, and gene expression data although any type of data can be submitted and shared.

# The Cancer Gene Trust Dameon
[The Cancer Gene Trust Daemon (cgtd)](https://github.com/cancergenetrust/cgtd) stores steward submissions in a distributed, replicated and decentralized content addressable database. It provides a basic HTML interface and RESTful API to add, list and authenticate submissions as well as the peering relationship between stewards. 

[search.cancergenetrust.org](http://search.cancergenetrust.org) is an example search engine across all current stewards that builds a searchable index with network viewer.

Submissions consist of a JSON manifest with a list of fields and files. Fields typically include de-identified clinical data (i.e. tumor type). Files typically consist of somatic variant vcf files and gene expression tsv file. Manifest's and files are stored and referenced by the [multihash](https://github.com/jbenet/multihash) of their content.

Eash steward has a top level JSON index file containing it's dns domain, list of submissions by multihash and list of peers by address. Each steward has a public private key pair which is used to authenticate their submissions. A steward's address is the multihash of their public key. A steward's address resolves to the multihash of the latest version of its index. Updates to a steward's index file are signed using their private key. This provides authentication and authorization for its contents as well as any other content referenced via multihash within it including all submissions. 

The current underlying implementation leverages [ipfs](http://ipfs.io) for storage and replication and ipns for address resolution and public/private key operations. The server is implemented using python and [flask](http://flask.pocoo.org/)

# Running a Production Instance

Note: The only dependencies for the following is make and docker-compose

Start cgtd and ipfs daemons with data stored in a docker volume:

    make up

Reset the steward's index to no submissions, no peers, set a domain:

    DOMAIN=lorem.edu make reset

To verify both cgtd and ipfs are working you can query your steward's address:

    curl localhost/v0/address

# Making Submissions

To make a test submission:

    make submit

or via curl:

    docker exec -it cgtd curl -X POST localhost:5000/v0/submissions \
        -F "a_field_name=a_field_value" \
        -F files[]=@tests/ALL/SSM-PAKMVD-09A-01D.vcf

To access the submission:

    curl localhost/v0/submissions/QmZwuc83iD64mvsf484aGcerUHJce1bJtf1y7AAzQDp234

Access control for mutable operations such as adding submissions or peers is restricted to localhost as a poor mans authentication. As a result we curl from within the cgtd container above.

To populate a server with a bunch of test data:

    make populate

Finally to see the index for you server including submissions:

    curl localhost/v0/submissions

# Build, Debug and Test Locally

Build a local cgtd docker container:

    make build

Start cgtd running using the local code with auto-reload:

    make debug

This runs the cgtd container listening on port 5000 out of the local folder so you can make changes and it will live reload.

To run tests open another terminal window and:

    make test
