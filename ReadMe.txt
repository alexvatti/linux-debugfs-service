debugfs helps kernel developers export large amounts of debug data into user-space.
It has no rules about any information that can be put in, unlike /proc and /sysfs,

Setting up DebugFS

If you are using one of the latest distributions, chances are that debugfs is already set up on your machine. If you’re compiling the kernel from scratch, make sure you enable debugfs in the kernel configuration. Once you reboot to your newly compiled kernel, check if debugfs is already mounted, with the following command:
# mount | grep  debugfs
none on /sys/kernel/debug type debugfs (rw)

ex:
mount | grep  debugfs
debugfs on /sys/kernel/debug type debugfs (rw,relatime)


If you see output as above, you have debugfs pre-mounted. If not, you can mount it (as root) with the command shown below:
# mount -t debugfs nodev /sys/kernel/debug


Code sample:

Note: You need to be running the latest kernel to make this program work.
This sample program, my_debugfs.c, shows a use of debugfs.


#include <linux/init.h> 
#include <linux/module.h> 
#include <linux/debugfs.h> /* this is for DebugFS libraries */ 
#include <linux/fs.h>   
MODULE_LICENSE("GPL"); 
MODULE_AUTHOR("Surya Prabhakar <surya_prabhakar@dell.com>"); 
MODULE_DESCRIPTION("sample code for DebugFS functionality"); 
#define len 200 
u64 intvalue,hexvalue; 
struct dentry *dirret,*fileret,*u64int,*u64hex; 
char ker_buf[len]; 
int filevalue; 
/* read file operation */
static ssize_t myreader(struct file *fp, char __user *user_buffer, 
                                size_t count, loff_t *position) 
{ 
     return simple_read_from_buffer(user_buffer, count, position, ker_buf, len);
} 
 
/* write file operation */
static ssize_t mywriter(struct file *fp, const char __user *user_buffer, 
                                size_t count, loff_t *position) 
{ 
        if(count > len ) 
                return -EINVAL; 
  
        return simple_write_to_buffer(ker_buf, len, position, user_buffer, count); 
} 
 
static const struct file_operations fops_debug = { 
        .read = myreader, 
        .write = mywriter, 
}; 
 
static int __init init_debug(void) 
{ 
    /* create a directory by the name dell in /sys/kernel/debugfs */
    dirret = debugfs_create_dir("dell", NULL); 
      
    /* create a file in the above directory 
    This requires read and write file operations */
    fileret = debugfs_create_file("text", 0644, dirret, &filevalue, &fops_debug);
 
    /* create a file which takes in a int(64) value */
    u64int = debugfs_create_u64("number", 0644, dirret, &intvalue); 
    if (!u64int) { 
        printk("error creating int file"); 
        return (-ENODEV); 
    } 
    /* takes a hex decimal value */
    u64hex = debugfs_create_x64("hexnum", 0644, dirret, &hexvalue ); 
    if (!u64hex) { 
        printk("error creating hex file"); 
        return (-ENODEV); 
    } 
 
    return (0); 
} 
module_init(init_debug); 
 
static void __exit exit_debug(void) 
{ 
    /* removing the directory recursively which 
    in turn cleans all the file */
    debugfs_remove_recursive(dirret); 
} 
module_exit(exit_debug);


Building the module:

Create a Makefile like the following one, to compile and test this code:

obj-m   := my_debugfs.o
KDIR    := /lib/modules/$(shell uname -r)/build
PWD     := $(shell pwd) 
 
all: 
                $(MAKE) -C $(KDIR) SUBDIRS=$(PWD) modules


DEBUG FS:
sudo ls -l /sys/kernel/debug/
total 0
drwxr-xr-x  2 root root 0 Mar 28 19:42 acpi
drwxr-xr-x 28 root root 0 Mar 28 19:42 bdi
drwxr-xr-x 24 root root 0 Mar 28 19:42 block
drwxr-xr-x  3 root root 0 Mar 28 19:42 bluetooth
drwxr-xr-x  2 root root 0 Mar 28 19:42 cec
drwxr-xr-x  2 root root 0 Mar 28 19:42 cleancache
--w-------  1 root root 0 Mar 28 19:42 clear_warn_once
drwxr-xr-x  4 root root 0 Mar 28 19:42 clk
drwxr-xr-x  2 root root 0 Mar 28 19:42 device_component
-r--r--r--  1 root root 0 Mar 28 19:42 devices_deferred
drwxr-xr-x  2 root root 0 Mar 28 19:42 dma_buf
drwxr-xr-x  4 root root 0 Mar 28 19:42 dri
drwxr-xr-x  2 root root 0 Mar 28 19:42 dynamic_debug
drwxr-xr-x  2 root root 0 Mar 28 19:42 error_injection
drwxr-xr-x  2 root root 0 Mar 28 19:42 extfrag
-rw-r--r--  1 root root 0 Mar 28 19:42 fault_around_bytes
drwxr-xr-x  2 root root 0 Mar 28 19:42 frontswap
-r--r--r--  1 root root 0 Mar 28 19:42 gpio
drwxr-xr-x  5 root root 0 Mar 28 19:42 hid
drwxr-xr-x  3 root root 0 Mar 28 19:42 ieee80211
drwxr-xr-x  4 root root 0 Mar 28 19:42 intel_lpss
drwxr-xr-x  2 root root 0 Mar 28 19:42 intel_powerclamp
drwxr-xr-x  2 root root 0 Mar 28 19:42 iosf_sb
drwxr-xr-x  3 root root 0 Mar 28 19:42 iwlwifi
drwxr-xr-x  2 root root 0 Mar 28 19:42 kprobes
drwxr-xr-x  2 root root 0 Mar 28 19:42 kvm
drwxr-xr-x  2 root root 0 Mar 28 19:42 mce
drwxr-xr-x  2 root root 0 Mar 28 19:42 mei0
-r--r--r--  1 root root 0 Mar 28 19:42 memcg_slabinfo
drwxr-xr-x  2 root root 0 Mar 28 19:42 mmc0
drwxr-xr-x  2 root root 0 Mar 28 19:42 opp
drwxr-xr-x  2 root root 0 Mar 28 19:42 pinctrl
drwxr-xr-x  2 root root 0 Mar 28 19:42 pkg_temp_thermal
drwxr-xr-x  2 root root 0 Mar 28 19:42 pmc_core
drwxr-xr-x  2 root root 0 Mar 28 19:42 pm_genpd
drwxr-xr-x  2 root root 0 Mar 28 19:42 pm_qos
-r--r--r--  1 root root 0 Mar 28 19:42 pwm
drwxr-xr-x  3 root root 0 Mar 28 19:42 ras
drwxr-xr-x  4 root root 0 Mar 28 19:42 regmap
drwxr-xr-x  3 root root 0 Mar 28 19:42 regulator
drwxr-xr-x  2 root root 0 Mar 28 19:42 remoteproc
-rw-r--r--  1 root root 0 Mar 28 19:42 sched_debug
-rw-r--r--  1 root root 0 Mar 28 19:42 sched_features
-r--r--r--  1 root root 0 Mar 28 19:42 sleep_time
--w-------  1 root root 0 Mar 28 19:42 split_huge_pages
-r--r--r--  1 root root 0 Mar 28 19:42 suspend_stats
drwxr-xr-x  2 root root 0 Mar 28 19:42 swiotlb
drwxr-xr-x  2 root root 0 Mar 28 19:42 sync
dr-xr-xr-x  3 root root 0 Mar 28 19:42 tracing
drwxr-xr-x  7 root root 0 Mar 28 19:42 usb
drwxr-xr-x  2 root root 0 Mar 28 19:42 virtio-ports
-r--r--r--  1 root root 0 Mar 28 19:42 wakeup_sources
drwxr-xr-x  2 root root 0 Mar 28 19:42 x86
drwxr-xr-x  2 root root 0 Mar 28 19:42 zswap

