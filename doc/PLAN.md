# Plan

## Library

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
│           │   ├── <job uuid>.json
│           |   └── ...
│           └── ...
└── ...
```

This gives us the ability to:

1. List jobs by a given certificate
1. Access to the details of a given job

## API

### mTLS support


