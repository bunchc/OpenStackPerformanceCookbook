# OpenStack Performance Tuning Cookbook

- Intro & High level what the book will cover
    - OpenStack Overview
    - Environment Overview
    - Methodology & Measuring
        + Ceilometer
    - On Security & Performance
        + Our decision making process
- Chapter 1 - Your System as a Whole
    + Hardware selection
    + Hardware tuning
    + Linux tuning
         (not sure if this naturally fits into other areas?)
    + Performance testing Linux
	   * tools/howto
- Chapter 2 - OpenStack Compute (Nova) Tuning
    + Ephemeral Disk Placement
    + KVM Tuning
        * Calculating Memory Overheads
    + QEMU Options
    + Libvirt
    + Overcommit
    + Host aggregates
    + Performance testing Nova
	* tools/howto
- Chapter 3 - Tuning Connective Tissue
    + Choosing a message queue
        * Tuning RabbitMQ
        * Tuning ZeroMQ
    + MySQL Tuning
- Chapter 4 - Tuning OpenStack Nova Network
    + Surplus, think remove.
- Chapter 5 - Tuning OpenStack Neutron
    + Network node
    + Kernel
    + TCP Stack
- Chapter 6 - Cinder Tuning
    + LVM
    + Storage backends
    + Encryption
- Chapter 7 - Keystone Tuning
    + memcache
    + Tuning LDAP Queries
    + APIs?
- Chapter 8 - Horizon Tuning
    + Apache
- Chapter 9 - Glance
    + With Registry/Without Registry
    + Scaling
        * Herd/Murder
    + Maintaing large catalogs
    + APIs?
- Chapter 10 - Instances (fit under Compute?)
    + guest tuning for clouds
    + can of worms? example stacks - tune for private cloud?
- Appendix - Quick Tips
    + Few pages of "standard" changes you expect to see in any install covering all areas?
    + Contribute @ website/github...
