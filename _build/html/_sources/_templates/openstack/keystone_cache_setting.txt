=======================================
keystone 配置token的cache
=======================================
keystone的Token可以存储在多种后端介质中，你可以存储在mysql中，也可以配置KVS存储，通常存储在mysql中的话，我们需要再配置一个cache，以便能够提高token相关请求的访问性能。


配置实现
=================
首先你要安装 apt-get install python-memcache memcached


配置keystone.conf中的[cache]模块如下::


    [cache]
    expiration_time=3600 #根据你需求更改

    backend=dogpile.cache.memcached
    backend_argument=url:localhost:11211

    enabled=true

我们这里使用的是memcached做缓存，你也可以使用redis、mongodb等等，token的cache是用dogpile.cache实现的，这里backend_argument也是兼容其配置的规范。

enabled一定要设为true，否则cache将不能使用。expiration_time是全局的失效时间， 其余模块如[token]和[assignment]都有cache_time属性，cache_time是各个模块的cache失效时间，cache_time如果不配置则默认expiration_time的值。




