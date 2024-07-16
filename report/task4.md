
```
~ # rmmod r4l_e1000_demo                                                                                                                                     
[  840.389601] r4l_e1000_demo: Rust for linux e1000 driver demo (exit)
[  840.390142] r4l_e1000_demo: Rust for linux e1000 driver demo (remove)
[  840.390397] r4l_e1000_demo: Rust for linux e1000 driver demo (device_remove)
[  840.391234] r4l_e1000_demo: Rust for linux e1000 driver demo (net device stop)
[  840.391633] r4l_e1000_demo: Rust for linux e1000 driver demo (net device get_stats64)
[  840.397383] r4l_e1000_demo: Rust for linux e1000 driver demo (net device get_stats64)
```

这些信息可以帮我们了解到调用顺序

```
~ # insmod r4l_e1000_demo.ko
[ 1078.377606] r4l_e1000_demo: Rust for linux e1000 driver demo (init)
[ 1078.378055] r4l_e1000_demo: Rust for linux e1000 driver demo (probe): None
[ 1078.378320] r4l_e1000_demo 0000:00:03.0: BAR 0: can't reserve [mem 0xfeb80000-0xfeb9ffff]
[ 1078.378605] r4l_e1000_demo: probe of 0000:00:03.0 failed with error -16
```

[C 语言版本 E1000 驱动](https://github.com/Rust-for-Linux/linux/blob/rust/drivers/net/ethernet/intel/e1000/e1000_main.c)

```c
// rust/drivers/net/ethernet/intel/e1000/e1000_main.c

static struct pci_driver e1000_driver = {
	.name     = e1000_driver_name,
	.id_table = e1000_pci_tbl,
	.probe    = e1000_probe,
	.remove   = e1000_remove,
	.driver = {
		.pm = &e1000_pm_ops,
	},
	.shutdown = e1000_shutdown,
	.err_handler = &e1000_err_handler
};

// ...

static void e1000_remove(struct pci_dev *pdev)
{
	struct net_device *netdev = pci_get_drvdata(pdev);
	struct e1000_adapter *adapter = netdev_priv(netdev);
	struct e1000_hw *hw = &adapter->hw;
	bool disable_dev;

	e1000_down_and_stop(adapter);
	e1000_release_manageability(adapter);

	unregister_netdev(netdev);

	e1000_phy_hw_reset(hw);

	kfree(adapter->tx_ring);
	kfree(adapter->rx_ring);

	if (hw->mac_type == e1000_ce4100)
		iounmap(hw->ce4100_gbe_mdio_base_virt);
	iounmap(hw->hw_addr);
	if (hw->flash_address)
		iounmap(hw->flash_address);
	pci_release_selected_regions(pdev, adapter->bars);

	disable_dev = !test_and_set_bit(__E1000_DISABLED, &adapter->flags);
	free_netdev(netdev);

	if (disable_dev)
		pci_disable_device(pdev);
}
```

```rust
// rust/kernel/pci.rs

impl<T: Driver> driver::DriverOps for Adapter<T> {
    type RegType = bindings::pci_driver;

    unsafe fn register(
        reg: *mut bindings::pci_driver,
        name: &'static CStr,
        module: &'static ThisModule,
    ) -> Result {
        // SAFETY: By the safety requirements of this function (defined in the trait definition),
        // `reg` is non-null and valid.
        let pdrv: &mut bindings::pci_driver = unsafe { &mut *reg };

        pdrv.name = name.as_char_ptr();
        pdrv.probe = Some(Self::probe_callback);
        pdrv.remove = Some(Self::remove_callback);
        pdrv.id_table = T::ID_TABLE.as_ref();
        // SAFETY:
        //   - `pdrv` lives at least until the call to `pci_unregister_driver()` returns.
        //   - `name` pointer has static lifetime.
        //   - `probe()` and `remove()` are static functions.
        //   - `of_match_table` is a raw pointer with static lifetime,
        //      as guaranteed by the [`driver::IdTable`] type.
        to_result(unsafe { bindings::__pci_register_driver(reg, module.0, name.as_char_ptr()) })
    }
}

// ...

impl<T: Driver> Adapter<T> {
    extern "C" fn remove_callback(pdev: *mut bindings::pci_dev) {
        // SAFETY: `pdev` is guaranteed to be a valid, non-null pointer.
        let ptr = unsafe { bindings::pci_get_drvdata(pdev) };
        // SAFETY:
        //   - we allocated this pointer using `T::Data::into_pointer`,
        //     so it is safe to turn back into a `T::Data`.
        //   - the allocation happened in `probe`, no-one freed the memory,
        //     `remove` is the canonical kernel location to free driver data. so OK
        //     to convert the pointer back to a Rust structure here.
        let data = unsafe { T::Data::from_pointer(ptr) };
        T::remove(&data);
        <T::Data as driver::DeviceRemoval>::device_remove(&data);
    }
}
```