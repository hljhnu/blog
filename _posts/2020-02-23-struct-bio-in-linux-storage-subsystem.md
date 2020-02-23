---
layout:post
title:Struct ```bio``` in Linux Storage Subsystem
---

The struct ```bio``` is a very import struct in linux storage subsystem. It is involved in many processes of I/O.

The struct ```bio``` (based on linux-4.4.88) is as below:

    struct bio {
		struct bio		*bi_next;	/* request queue link */
		struct block_device	*bi_bdev;
		unsigned int		bi_flags;	/* status, command, etc */
		int			bi_error;
		unsigned long		bi_rw;		/* bottom bits READ/WRITE,
							 * top bits priority
							 */
	
		struct bvec_iter	bi_iter;
	
		/* Number of segments in this BIO after
		 * physical address coalescing is performed.
		 */
		unsigned int		bi_phys_segments;
	
		/*
		 * To keep track of the max segment size, we account for the
		 * sizes of the first and last mergeable segments in this bio.
		 */
		unsigned int		bi_seg_front_size;
		unsigned int		bi_seg_back_size;
	
		atomic_t		__bi_remaining;
	
		bio_end_io_t		*bi_end_io;
	
		void			*bi_private;
		#ifdef CONFIG_BLK_CGROUP
		/*
		 * Optional ioc and css associated with this bio.  Put on bio
		 * release.  Read comment on top of bio_associate_current().
		 */
		struct io_context	*bi_ioc;
		struct cgroup_subsys_state *bi_css;
	    #endif
		union {
	    #if defined(CONFIG_BLK_DEV_INTEGRITY)
			struct bio_integrity_payload *bi_integrity; /* data integrity */
	    #endif
		};
	
		unsigned short		bi_vcnt;	/* how many bio_vec's */
	
		/*
		 * Everything starting with bi_max_vecs will be preserved by bio_reset()
		 */
	
		unsigned short		bi_max_vecs;	/* max bvl_vecs we can hold */
	
		atomic_t		__bi_cnt;	/* pin count */
	
		struct bio_vec		*bi_io_vec;	/* the actual vec list */
	
		struct bio_set		*bi_pool;
	
		/*
		 * We can inline a number of vecs at the end of the bio, to avoid
		 * double allocations for a small number of bio_vecs. This member
		 * MUST obviously be kept at the very end of the bio.
		 */
		struct bio_vec		bi_inline_vecs[0];
    };

A ```bio``` is used for a continuous data transter between device and memory. The device addresses of a ```bio``` is continuous in the device. Although device addresses of a ```bio``` is continuous, the corressponding memory addresses are not neccessarily continous. A struct ```bio_vec``` is used to represent the memory blocks related to the memory addresses.

The struct ```bio_vec``` is as below:

	/*
	 * was unsigned short, but we might as well be ready for > 64kB I/O pages
	 */
	struct bio_vec {
		struct page	*bv_page;
		unsigned int	bv_len;
		unsigned int	bv_offset;
	};

When a ```bio``` is created, there is no valid ```bio_vec```. ```bio_vec``` can be added to the ```bio``` later, for example, the function ```bio_add_page``` is used to do it. ```bio_for_each_segment``` can be used to iterate all ```bio_vec```s in a ```bio```. Struct bio is widely used in file system. When a file system needs to read or write data, it may prepare a ```bio``` and then call ```submit_bio``` to perform the read or write operation. For example, F2FS calls ```submit_bio``` in ```f2fs_mpage_readpages``` to read data from devices to memory pages.

