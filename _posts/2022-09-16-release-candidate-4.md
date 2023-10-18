---
layout: post
status: publish
published: true
title: PocketSphinx 5.0.0 release candidate 4
author:
  display_name: David Huggins-Daines
  email: dhdaines@gmail.com
author_email: dhdaines@gmail.com
excerpt_separator: <!--more-->
---

**Executive Summary: Alas, poor SphinxBase!**

Yes, it's that time of week again, time for another [release
candidate](https://github.com/cmusphinx/pocketsphinx/releases/tag/v5.0.0rc4).
You can also download it [from
PyPI](https://pypi.org/project/pocketsphinx/).

In the spirit of total elimination, the major change here is the
disappearance of the `<sphinxbase/*.h>` headers.  Some of them have
been relocated, so if you include `<pocketsphinx.h>` you can still do
useful things like load and save language models and parse JSGF.  Oh,
and also do speech recognition, maybe.

There are a number of other things you can't do, because the "utility"
headers were mostly unsuitable for public consumption.  Really they
were a bit embarrassing, at least in 2022.  A major rationale for
removing SphinxBase from circulation is that it just isn't a good
foundation for you to build "applications" or anything else really.
Like, there are at least a dozen better implementations of pretty much
everything in there, and you should really use them.  Command-line
parsing, for instance, should *not* be done with `<cmd_ln.h>`, so it
has been hidden from you to discourage you from trying that.

Which brings us to the other major breaking change here.
Configuration is not done by parsing (possibly imaginary) command
lines anymore.  You can simply create a configuration and set values
in it, e.g.:

    ps_config_t *config = ps_config_init(NULL);
    ps_config_set_str(config, "hmm", "/path/to/model");
    ps_config_set_int(config, "samprate", 11025);

You can also parse JSON, or even a sort of degenerate "JSON":

    ps_config_t *config = ps_config_parse_json(
        NULL, "{\"hmm\": \"/path/to/model\"}");
    ps_config_t *config = ps_config_parse_json(
        NULL, "hmm: /path/to/model, samprate: 11025");

The configuration can be serialized to (actual) JSON as well:

    const char *jconf = ps_config_serialize_json(config);

Creating a `ps_config_t` sets all of the default values, but does not
set the default model, so you still need to use
`ps_default_search_args()` for that.  Also note that
`ps_expand_model_config()` no longer creates magical underscore
versions of the config parameters (e.g. `"_hmm"`, `"_dict"`, etc) but
simply overwrites the existing values.

Python code is entirely unaffected by these changes (though it has
also acquired the JSON functions mentioned above), so you should maybe
use Python instead of hurting yourself with the C API.

Pull requests and bug reports and such are welcome via
[GitHub](https://github.com/cmusphinx/pocketsphinx).
