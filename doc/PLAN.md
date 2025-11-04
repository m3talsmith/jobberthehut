# Plan

## Library

### Features

The library will have the following features in the scope of this project:

1. Provide access to the certificates from a configuration
1. Manage jobs

### Provide access to the certificates from a configuration

This is a straightforward as reading the pools from the storage and providing them via a `func GetClientCertificates() ([]*Certificate, ErrorCertificate)` function.

If there are no certificatetes, it returns an empty slice of `*Certificate`.

If there was an error (e.g. read permissions issue) it returns a `ErrorCertificate` error.

### Manage jobs

In order to manage the jobs, the library will expose the following functions:

- `func NewCertificate(pem []byte) (cert *Certificate, err ErrorCertificate)`: instantiate a certificate from a pem
- `func (*Certificate) Bytes() (pem []byte)`: return the pem of a certificate
- `func (*Certificate) Hash() (hash string)`: sha256 hash the contents of a certificate
- `func (*Certificate) GetJobs() (jobs []*Job, err ErrorCertificate)`: get a list of jobs
- `func (*Certificate) GetJob(id int) (job *Job, err ErrorCertificate)`: get a job from id
- `func (*Certificate) GenerateJobID() (id int)`: generate the next ID for a job
- `func (*Certificate) NewJob(cmd string) (job *Job, err ErrorCertificate)`: instantiate a job with a command
- `func (*Job) Start() (err ErrorJob)`: start a job
- `func (*Job) Tail(readTo []byte, offset int64) (err ErrorJob)`: read a given amount of lines from the end of the log
- `func (*Job) Stop() (err ErrorJob)`: stop a job

#### NewCertificate

The `NewCertificate` function takes a `[]byte` representation of a certificate pem.

It checks to see if the certificate is valid (e.g. by parsing it, looking at the certificate pool in storage, etc.) and returns a `*Certificate` if true. If false, it returns an `ErrorCertificate`.

#### Bytes

The `(*Certificate) Bytes` function returns the pem of a certificate as a `[]byte`.

#### Hash

The `(*Certificate) Hash` function creates a sha256 hash of the certificate contents and returns that as a `string`

#### GetJobs

The function `(*Certificate) GetJobs` returns a list of jobs for a `*Certificate` hash in storage. If there is an error (e.g. problem with read permissions) it will return an `ErrorCertificate`.

#### GetJob

The `(*Certificate) GetJob` function takes an ID as a `int`.

It operates by looking up the job for a `*Certificate` hash in storage. If it is found it returns a `*Job`, otherwise it returns an `ErrorCertificate`.

#### GenerateJobID

The `(*Certificate) GenerateJobID` function generates a new ID for a future `*Job`. It does this by looking at the length of of the return from `GetJobs` and appending a Unix timestamp to it.

If it is successful, it returns the ID as an `int`. If there are no jobs returned, it defaults to 1 plus the Unix timestamp. If there was an error (e.g. read permissions issue in storage) it returns an `ErrorCertificate`.

#### NewJob

The `(*Certificate) NewJob` function attempts to create a job with a command for the certificate.

This instantiates a `*Job` with a generated ID (using `GenerateJobID`) and returns it. If it cannot generate the ID due to some error, it will return the `ErrorCertificate`.

#### Start

The `(*Job) Start` function attempts to start a `*Job` for a `*Certificate` hash in storage.

If the hash exists as a directory in storage, then we:

1. Create a directory of the jobID in the certificate directory
1. Create a cgroup for the job
1. Execute the job in a cgroup with output redirected to a log in the job directory
1. Write the job pid to the job directory

If there an error, it returns an `ErrorJob`.

#### Tail

The `(*Job) Tail` function takes a variable to write to as `[]byte` and attempts to read the job log. If it can read the log, it writes it to this variable. If it cannot read the log, it returns an `ErrorJob`.

#### Stop

The `(*Job) Stop` function attempts to stop the job. It does this by:

1. Reading the pid for the job
1. Killing the process
1. Removing the cgroup

If there is an error, then a `ErrorJob` is returned.

### Storage

#### Configuration

The configuration will be in this structure:

```
<root>
├── pool
│   ├── <certificate>
│   └── ...
└── ...
```

This allows us to store certs in a pool to be used at a future point in time.

#### Jobs

The jobs will be written as ephemeral to this structure:

```
/
├── tmp
│   └── jobber
│       └── jobs
│           ├── <sha256 hash of certificate>
│           │   ├── <job uuid>
│           |   |   ├── pid
│           |   |   └── log
│           |   └── ...
│           └── ...
└── ...
```

This gives us the ability to:

1. List jobs by a given certificate
1. Access to the details of a given job

## API

### mTLS support


