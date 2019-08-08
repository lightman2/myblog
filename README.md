					Setup_arch
					bootmem_init();   bootmem分配器  用于在启动阶段早期分配内存 
						zone_sizes_init(min, max);  UMA和NUMA同名
							free_area_init_nodes(max_zone_pfns); 架构无关  NUMA
							free_area_init_node(0, zone_size, min, zhole_size);（架构无关UMA少个S！！）
								alloc_node_mem_map  dgl_note!!!!
										NODE_DATA node_data----------------NUMA
										#define NODE_DATA(nid)	(&contig_page_data)
											struct pglist_data __refdata contig_page_data;
									mem_map == (struct pglist_data *pgdat)NODE_DATA(0)->node_mem_map;
									struct page *mem_map;   !!
										内核会将所有struct page* 放到一个全局数组（mem_map）中，而内核中我们常会看到pfn，说得就是页帧号，也就是数组的index，这里的MAX_ARCH_PFN就是系统的最大页帧号，但这个只是理论上的最大值，在start_kernel()时，setup_arch()函数会通过e820_end_of_ram_pfn()函数来获得实际物理内存并返回最终的max_pfn，可以看下e820_end_of_ram_pfn的实现（其内部直接调用e820_end_pfn函数）
										
										来自 <https://www.cnblogs.com/emperor_zark/archive/2013/03/15/linux_page_1.html> 
										
										
