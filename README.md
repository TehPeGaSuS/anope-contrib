# anope-contrib

Third-party Anope modules and patches, adapted/ported for Anope 2.1, used on [hookupz.win](https://hookupz.win).

## Modules

### `modules/hs_nethost.cpp`

Ports [Techman's `hs_nethost`](https://gist.github.com/7330c12ce7a03c030871) (HostServ module that assigns a default vhost based on a user's account name) to the Anope 2.1 API.

Original module targeted Anope 2.0's API (`anope_override`, lowercase `Vhost` accessors, pointer-based `OnReload`). This port updates it to 2.1's API: `override`, `SetVHost`/`HasVHost`/`GetVHostCreator`/`GetVHostIdent` (capitalized), the `OnSetVHost` hook, reference-based `OnReload(Configuration::Conf &conf)`, and modern range-based loops.

It also beautifies vhosts generated from nicks with heavy symbol use. The original module replaced every character not in `[A-Za-z0-9-]` with a literal `-`, so a nick like `|||Foo|||BAr|||` produced a vhost with ugly runs of dashes (`user/---Foo---BAr---/x-<hash>`). This port collapses consecutive dashes and trims leading/trailing ones, so the same nick now yields `user/Foo-BAr/x-<hash>` — still hash-suffixed (since the nick still contained invalid characters, so collisions with other symbol-heavy nicks are still avoided), just readable. If a nick is made up entirely of invalid characters, it falls back to `user` (e.g. `user/user/x-<hash>`) instead of producing a vhost with nothing but the bare prefix/suffix.

`valid_nick_chars` in this fork is `[A-Za-z0-9_-]` (adds `_` to the original's `[A-Za-z0-9-]`), since UnrealIRCd's own hostname validation (`valid_host()`) already allows underscores. **If you enable this**, Anope's own vhost validator (`IRCDProto::IsHostValid()`, which `hs_nethost` calls before setting a vhost) also needs to allow `_`, or valid underscore-containing vhosts will be rejected as invalid. Set in `anope.conf`:
```
networkinfo
{
	...
	vhost_chars = "abcdefghijklmnopqrstuvwxyzABCDEFGHIJKLMNOPQRSTUVWXYZ0123456789_.-/";
	...
};
```

Configuration (`anope.conf`):
```
module { name = "hs_nethost"; prefix = "user/"; suffix = ""; hashprefix = "/x-"; setifnone = true; }
command { service = "HostServ"; name = "SETNETHOSTS"; command = "hostserv/setnethosts"; permission = "hostserv/setnethosts"; }
```

### Recommended: UnrealIRCd spamfilter to enforce the same charset on nick registration

`hs_nethost` mangles/hashes nicks with unsupported characters rather than rejecting them, so users can still register nicks like `|||Foo|||` — they just get an uglier, hash-suffixed vhost. To reject such nicks outright at registration/nick-change time instead, add this to UnrealIRCd's config (e.g. `spamfilter.conf`):
```
spamfilter {
	match-type regex;
	match '^[^!]*[^A-Za-z0-9_-][^!]*!';
	target user;
	action block;
	reason "Nicknames may only contain letters, digits, - and _ (please choose a different nickname)";
}
```
`target user;` matches the `nick!user@host:realname` string built on every `NICK` command (connect + change; see `src/modules/nick.c` and `src/modules/tkl.c`'s `_spamfilter_build_user_string()`). The regex flags any character before the first `!` (i.e. in the nick) that isn't in `[A-Za-z0-9_-]`. `action block;` rejects the command without killing/banning the user. IRC operators with the `immune:server-ban:spamfilter` permission are exempt by default.

## License

Each module retains the license of its original author; see the header comment in each file.
