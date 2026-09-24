# anope-contrib

Third-party Anope modules and patches, adapted/ported for Anope 2.1, used on [hookupz.win](https://hookupz.win).

## Modules

### `modules/hs_nethost.cpp`

Ports [Techman's `hs_nethost`](https://gist.github.com/7330c12ce7a03c030871) (HostServ module that assigns a default vhost based on a user's account name) to the Anope 2.1 API.

Original module targeted Anope 2.0's API (`anope_override`, lowercase `Vhost` accessors, pointer-based `OnReload`). This port updates it to 2.1's API: `override`, `SetVHost`/`HasVHost`/`GetVHostCreator`/`GetVHostIdent` (capitalized), the `OnSetVHost` hook, reference-based `OnReload(Configuration::Conf &conf)`, and modern range-based loops.

Configuration (`services.conf`):
```
module { name = "hs_nethost"; prefix = "user/"; suffix = ""; hashprefix = "/x-"; setifnone = true; }
command { service = "HostServ"; name = "SETNETHOSTS"; command = "hostserv/setnethosts"; permission = "hostserv/setnethosts"; }
```

## License

Each module retains the license of its original author; see the header comment in each file.
