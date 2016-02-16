# MessagePack Erlang

[![Travis](https://secure.travis-ci.org/msgpack/msgpack-erlang.png)](https://travis-ci.org/msgpack/msgpack-erlang)

[![Drone.io](https://drone.io/github.com/msgpack/msgpack-erlang/status.png)](https://drone.io/github.com/msgpack/msgpack-erlang)

## Prerequisites for runtime

[Erlang/OTP](http://erlang.org/), >= 17.0 Also based on
[the new msgpack spec 0b8f5a](https://github.com/msgpack/msgpack/blob/0b8f5ac67cdd130f4d4d4fe6afb839b989fdb86a/spec.md).

## edit rebar.config to use in your application

```erlang
{deps, [
  {msgpack, ".*",
    {git, "git://github.com/msgpack/msgpack-erlang.git", {branch, "master"}}}
]}.
```

## Simple deserialization

```erlang
Ham = msgpack:pack(Spam),
{ok, Spam} = msgpack:unpack(Ham).
```

## Stream deserialization

```erlang
{Term0, Rest0} = msgpack:unpack_stream(Binary),
{Term1, Rest1} = msgpack:unpack_stream(Rest0),
...
```

## Options, for packing and unpacking

### `{spec, new|old}`

Both for packing and unpacking. Major difference between old and new spec is:

- raw family (`0xa0~0xbf`, `0xda`, `0xdb`) becomes new str family
- `0xd9` is new as str8
- new bin space (`0xc4, 0xc5, 0xc6` as bin8, bin16, bin32)
- new ext space (`0xc7, 0xc8, 0xc9` as ext8, ext16, ext32)
- new fixext space (`0xd4, 0xd5, 0xd6, 0xd7, 0xd8` as fixext1, fixext2, fixext4, fixext8, fixext16),

The default is new spec. Old spec mode does not handle these new types but
returns error. To use
[old spec](https://github.com/msgpack/msgpack/blob/master/spec-old.md)
mode, this option is explicitly added.

```erlang
OldHam = msgpack:pack(Spam, [{spec, old}]),
{ok, Spam} = msgpack:unpack(OldHam, [{spec, old}]).
```

### `{allow_atom, none|pack}`

Only in packing. Atoms are packed as binaries.

### `{known_atoms, [atom()]}`

Both in packing and unpacking. In packing, if an atom is in this list
a binary is encoded as a binary. In unpacking, msgpacked binaries are
decoded as atoms with `erlang:binary_to_existing_atom/2` with encoding
`utf8`.

### `{str, as_binary|as_list}`

Both in packing and unpacking. Only available at new spec.

This option indicates str type family is treated as binary or list in
Erlang world.

```
mode        as_binary    as_list
-----------+------------+-------
packing
binary()    str          bin
string()    bin*         str*
list()      array        array
-----------+------------+-------
unpacking
bin         string()     binary()
str         binary()     string()
```

- (\*) fallback to list() handling if any error found


```erlang
Opt = [{enable_str, true}]
{ok, "埼玉"} = msgpack:unpack(msgpack:pack("埼玉", Opt), Opt).
 => {ok,[22524,29577]}
```
### `{validate_string, boolean()}`

Both in packing and unpacking. UTF-8 validation in encoding to str and
decoding from str type will be enabled.

### `{map_format, maps|jiffy|jsx}`

Both at packing and unpacking.

```erlang
msgpack:pack(#{ <<"key">> => <<"value">> }, []).
msgpack:pack({[{<<"key">>, <<"value">>}]}, [{format, jiffy}]),
msgpack:pack([{<<"key">>, <<"value">>}], [{format, jsx}]).
```


### `{ext, {msgpack_ext_packer(), msgpack_ext_unpacker()}|module()}`

At both.

Now msgpack-erlang supports ext type. Now you can serialize everything
with your original (de)serializer. That will enable us to handle
erlang- native types like `pid()`, `ref()` contained in `tuple()`. See
`test/msgpack_ext_example_tests.erl` for example code.

```erlang
Packer = fun({ref, Ref}, Opt) when is_reference(Ref) -> {ok, {12, term_to_binary(Ref)}} end,
Unpacker = fun(12, Bin) -> {ok, {ref, binary_to_term(Bin)}} end,
Ref = make_ref(),
Opt = [{ext,{Packer,Unpacker}}],
{ok, {ref, Ref}} = msgpack:unpack(msgpack:pack({ref, Ref}, Opt), Opt).
```

## License

Apache License 2.0

# Release Notes

## 0.5.0

- Renewed optional arguments to pack/unpack interface. This is
  incompatible change from 0.4 series.

## 0.4.0

- Deprecate `nil`
- Moved to rebar3
- Promote default map unpacker as default format when OTP is >= 17
- Added QuickCheck tests
- Since this version OTP older than R16B03-1 are no more supported

## 0.3.5 / 0.3.4

- 0.3 series will be the last versions that supports R16B or older
  versions of OTP.
- OTP 18.0 support
- Promote default map unpacker as default format when OTP is >= 18

## 0.3.3

- Add OTP 17 series to Travis-CI tests
- Fix wrong numbering for ext types
- Allow packing maps even when {format,map} is not set
- Fix Dialyzer invalid contract warning
- Proper use of null for jiffy-style encoding/decoding

## 0.3.2

- set back default style as jiffy
- fix bugs around nil/null handling

## 0.3.0

- supports map new in 17.0
- jiffy-style maps will be deprecated in near future
- set default style as map

## 0.2.8

0.2 series works with OTP 17.0, R16, R15, and with MessagePack's new
and old format. But does not support `map` type introduced in
OTP 17.0.

It also supports JSX-compatible mode.
