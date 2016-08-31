# NAME

CHI::Driver::Redis::SortedSet - Redis driver for CHI with proper expiration of namespace keys

# SYNOPSIS

    use CHI;

    my $cache = CHI->new(
        driver    => 'Redis::SortedSet',
        namespace => 'products',
        server    => '127.0.0.1:6379',
        debug     => 0
    );

# DESCRIPTION

Extends [CHI::Driver::Redis](https://metacpan.org/pod/CHI::Driver::Redis) to address memory leak issues from an unbound
`set` holding the namespace members. This module implements the fix as
suggested in a feature request by Pieter Noordhuis as outlined
[here](https://github.com/antirez/redis/issues/135).

The expiration mechanism is implemented as a lazy cleanup, transparently
invoked everytime `store` is called via `CHI::set()`.

Please note that this is **not** backwards compatible with existing Redis
datasets that have already been populated with entries via the
[CHI::Driver::Redis](https://metacpan.org/pod/CHI::Driver::Redis) module due to the underlying change in the data type
holding the list of all keys in the namespace as retrieved thru
`CHI::get_keys()`.

# FAQ

### I'm starting fresh with a new Redis database. What should I do?

Nothing.

Congratulations! You're in the best position to use this module instead of
[CHI::Driver::Redis](https://metacpan.org/pod/CHI::Driver::Redis) if memory usage can become an issue based on your setup
and use case.

### I've been using [CHI::Driver::Redis](https://metacpan.org/pod/CHI::Driver::Redis). How do I migrate to this new module?

The [FLUSHDB](http://redis.io/commands/FLUSHDB) command should first be
issued. Please note that this will drop all existing keys and cached data
and so is a destructive procedure.

### Why not make the driver backwards compatible with [CHI::Driver::Redis](https://metacpan.org/pod/CHI::Driver::Redis)?

Yes, that's very much possible. And should likewise not be too costly as
it will only have to involve a one-time transparently-invoked migration to
migrate all previously-defined namespace members from the original `set`
to a `sorted set`, with expiration values set from each key's
[TTL](http://redis.io/commands/TTL).

However, it would be prudent that the memory leak is an actual issue being
experienced by the developer before using this module. At this point, the
developer should first be aware of the subtle changes which will happen
under the hood.

As such, the author has made a judgment call to make this a conscious
decision on the developer's part.

### Why not send a pull request and incorporate this into [CHI::Driver::Redis](https://metacpan.org/pod/CHI::Driver::Redis)?

The author will first collaborate with the authors of [CHI::Driver::Redis](https://metacpan.org/pod/CHI::Driver::Redis)
and if deemed acceptable, merge the modifications into the original code
base so we won't have to deal with the confusion of having multiple [CHI](https://metacpan.org/pod/CHI)
drivers doing pretty much the same thing.

Otherwise, this module will continue to exist on a different namespace to
provide developers with this option.

# BUGS

`CHI::purge()` is **not** implemented, just like [CHI::Driver::Redis](https://metacpan.org/pod/CHI::Driver::Redis).

# AUTHOR

Arnold Tan Casis <atancasis@cpan.org>

# ACKNOWLEDGMENTS

This is based on the work by Cory G Watson &lt;gphat at cpan.org> and Ian
Burrell <iburrell@cpan.org> so all attributions should go to them.

Likewise, the fix implemented in this module is inspired by the suggestion
of Pieter Noordhuis <pcnoordhuis@gmail.com>.

# COPYRIGHT

Copyright 2016- Arnold Tan Casis

# LICENSE

This library is free software; you can redistribute it and/or modify
it under the same terms as Perl itself.

# SEE ALSO

[CHI::Driver::Redis](https://metacpan.org/pod/CHI::Driver::Redis)
