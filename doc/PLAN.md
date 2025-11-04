# Plan

## Library

### Storage

The configuration will be in this structure:

```
<root>
├── pools
│   ├── <sha 256 hash of certificate>
│   └── ...
└── ...
```

The logs will be written as ephemeral to this structure:

```
/
├── tmp
│   └── jobber
│       └── jobs
│           ├── <sha256 hash of certificate>
│           │   ├── <job uuid>.log
│           │   |   ├── pid
│           │   │   └── log
│           |   └── ...
│           └── ...
└── ...
```

## API

### mTLS support


