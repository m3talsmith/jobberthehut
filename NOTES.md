# Jobber The Hut Notes

## Security concerns

- Authentication: using mTLS
    - CA
        - Cryptography
            - Strength
                - 1024
                - 2048
            - Algorithm
                - RSA
                - DSA
                - Ecliptic Curve
        - Generation
            - Assumed self-signed for MVP sake
        - Storage
            - Location
            - Exposure
        - Issuance
        - Verification
- Isolation: where do we run the jobs?
    - In a namespace
    - In chroot

### mTSL tooling and libraries

There are some tools we may want to create to handle mTLS:

- A library to generate the CRT and CA
- A cli to generate the CRT and CA for the server

## Control groups

### Virtual resources

Given the nature of supporting arbitrary execution of code, it might be safe to allow for things like:

- Concurrency: allowing for multiple threads to run across two or more cores
- In-memory: allowing for a decent size of memory for processing
- Storage: allowing for a light storage footprint for sharing between processes

#### Constraints

The constraints listed below allow for at least two full threads for concurrency, a small database in memory, and room for plenty of configs, an SQLite db, and miscellaneous storage needs.

- CPU: 2 cores
- MEM: 8 GB
- HHD: 256 MB

### Job considerations

#### Resource management

- Create a cgroup per user with total virtual resources
- Create a cgroup per job under the user cgroup for shared virtual resources
- Create a cgroup per job with total virtual resources

#### Possible error / exit events to handle and cleanup

- cgroup runs out of virtual resources and exits
- OOM kills process inside of cgroup due to lack of real resources
- process inside cgroup errors and exits
- process inside cgroup completes and exits

#### Artifacts

The handling of artifacts, as a result of a job, may be handled with these options:

- Store artifacts
    - In a flat directory created for the user's private key
    - In a job directory created in the user's private key directory
    - Named using a timestamp of when the job was created
    - Named using the jobs pid
- Discard artifacts

#### Cleanup

There will be multiple things to cleanup when a job is done:

- Any temporary data created by a job
- The cgroup if cgroups are job based
- All related processes either direct or forked from the job