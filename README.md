# Rust-lesson6

### 下载node template

`git clone -b v2.0.0-rc5 --depth 1 https://github.com/substrate-developer-hub/substrate-node-template`



### 编译

`Cargo build --release`



### 执行

`./target/release/node-template --dev`



### 使用polkadot js

https://polkadot.js.org/apps/#/explorer

### 创建凭证

```rust
		#[weight = 0]  // 权重
		// 创建凭证
		pub fn create_claim(origin, claim: Vec<u8>) -> dispatch::DispatchResult {
			let sender = ensure_signed(origin)?;
			ensure!(!Proofs::<T>::contains_key(&claim), Error::<T>::ProofsAlreadyExists); // 验证是否存在
			Proofs::<T>::insert(&claim, (sender.clone(), frame_system::Module::<T>::block_number()));
			// 触发事件
			Self::deposit_event(RawEvent::ClaimCreated(sender, claim));
			Ok(())
		}
```



### 撤销凭证

```rust
#[weight = 0]  // 权重
		// 吊销凭证
		pub fn revoke_claim(origin, claim: Vec<u8>) -> dispatch::DispatchResult {
			let sender = ensure_signed(origin)?;
			ensure!(!Proofs::<T>::contains_key(&claim), Error::<T>::ClaimNotExists);
			// 获取当前claim
			let (owner, _block_number) = Proofs::<T>::get(&claim);
			// 校验发送方 owner
			ensure!(owner == sender, Error::<T>::NotClaimOwner);
			Proofs::<T>::remove(&claim);
			Self::deposit_event(RawEvent::ClaimRevoked(sender, claim));
			Ok(())
		}
```



runtime中实现接口

```rust
impl poe::Trait for Runtime {
	type Event = Event;
}
```



```rust
construct_runtime!(
	pub enum Runtime where
		Block = Block,
		NodeBlock = opaque::Block,
		UncheckedExtrinsic = UncheckedExtrinsic
	{
		System: system::{Module, Call, Config, Storage, Event<T>},
		RandomnessCollectiveFlip: randomness_collective_flip::{Module, Call, Storage},
		Timestamp: timestamp::{Module, Call, Storage, Inherent},
		Aura: aura::{Module, Config<T>, Inherent},
		Grandpa: grandpa::{Module, Call, Storage, Config, Event},
		Balances: balances::{Module, Call, Storage, Config<T>, Event<T>},
		TransactionPayment: transaction_payment::{Module, Storage},
		Sudo: sudo::{Module, Call, Config<T>, Storage, Event<T>},
		// Include the custom logic from the template pallet in the runtime.
		TemplateModule: template::{Module, Call, Storage, Event<T>},
		PoeModule: poe::{Module, Call, Storage, Event<T>},
	}
);
```

