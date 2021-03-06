
<a name="0x1_DesignatedDealer"></a>

# Module `0x1::DesignatedDealer`

### Table of Contents

-  [Resource `Dealer`](#0x1_DesignatedDealer_Dealer)
-  [Struct `ReceivedMintEvent`](#0x1_DesignatedDealer_ReceivedMintEvent)
-  [Function `publish_designated_dealer_credential`](#0x1_DesignatedDealer_publish_designated_dealer_credential)
-  [Function `add_tier_`](#0x1_DesignatedDealer_add_tier_)
-  [Function `add_tier`](#0x1_DesignatedDealer_add_tier)
-  [Function `update_tier_`](#0x1_DesignatedDealer_update_tier_)
-  [Function `update_tier`](#0x1_DesignatedDealer_update_tier)
-  [Function `tiered_mint_`](#0x1_DesignatedDealer_tiered_mint_)
-  [Function `tiered_mint`](#0x1_DesignatedDealer_tiered_mint)
-  [Function `exists_at`](#0x1_DesignatedDealer_exists_at)
-  [Function `reset_window`](#0x1_DesignatedDealer_reset_window)
-  [Function `window_length`](#0x1_DesignatedDealer_window_length)
-  [Specification](#0x1_DesignatedDealer_Specification)
    -  [Function `add_tier`](#0x1_DesignatedDealer_Specification_add_tier)
    -  [Function `update_tier`](#0x1_DesignatedDealer_Specification_update_tier)
    -  [Function `tiered_mint`](#0x1_DesignatedDealer_Specification_tiered_mint)
    -  [Function `exists_at`](#0x1_DesignatedDealer_Specification_exists_at)



<a name="0x1_DesignatedDealer_Dealer"></a>

## Resource `Dealer`



<pre><code><b>resource</b> <b>struct</b> <a href="#0x1_DesignatedDealer_Dealer">Dealer</a>
</code></pre>



<details>
<summary>Fields</summary>


<dl>
<dt>

<code>window_start: u64</code>
</dt>
<dd>
 Time window start in microseconds
</dd>
<dt>

<code>window_inflow: u64</code>
</dt>
<dd>
 The minted inflow during this time window
</dd>
<dt>

<code>tiers: vector&lt;u64&gt;</code>
</dt>
<dd>
 0-indexed array of tier upperbounds
</dd>
<dt>

<code>mint_event_handle: <a href="Event.md#0x1_Event_EventHandle">Event::EventHandle</a>&lt;<a href="#0x1_DesignatedDealer_ReceivedMintEvent">DesignatedDealer::ReceivedMintEvent</a>&gt;</code>
</dt>
<dd>
 Handle for mint events
</dd>
</dl>


</details>

<a name="0x1_DesignatedDealer_ReceivedMintEvent"></a>

## Struct `ReceivedMintEvent`



<pre><code><b>struct</b> <a href="#0x1_DesignatedDealer_ReceivedMintEvent">ReceivedMintEvent</a>
</code></pre>



<details>
<summary>Fields</summary>


<dl>
<dt>

<code>destination_address: address</code>
</dt>
<dd>

</dd>
<dt>

<code>amount: u64</code>
</dt>
<dd>

</dd>
</dl>


</details>

<a name="0x1_DesignatedDealer_publish_designated_dealer_credential"></a>

## Function `publish_designated_dealer_credential`



<pre><code><b>public</b> <b>fun</b> <a href="#0x1_DesignatedDealer_publish_designated_dealer_credential">publish_designated_dealer_credential</a>(dd: &signer, tc_account: &signer)
</code></pre>



<details>
<summary>Implementation</summary>


<pre><code><b>public</b> <b>fun</b> <a href="#0x1_DesignatedDealer_publish_designated_dealer_credential">publish_designated_dealer_credential</a>(
    dd: &signer,
    tc_account: &signer,
) {
    <b>assert</b>(<a href="Roles.md#0x1_Roles_has_treasury_compliance_role">Roles::has_treasury_compliance_role</a>(tc_account), EACCOUNT_NOT_TREASURY_COMPLIANCE);
    move_to(
        dd,
        <a href="#0x1_DesignatedDealer_Dealer">Dealer</a> {
            window_start: <a href="LibraTimestamp.md#0x1_LibraTimestamp_now_microseconds">LibraTimestamp::now_microseconds</a>(),
            window_inflow: 0,
            tiers: <a href="Vector.md#0x1_Vector_empty">Vector::empty</a>(),
            mint_event_handle: <a href="Event.md#0x1_Event_new_event_handle">Event::new_event_handle</a>&lt;<a href="#0x1_DesignatedDealer_ReceivedMintEvent">ReceivedMintEvent</a>&gt;(dd),
        }
    )
}
</code></pre>



</details>

<a name="0x1_DesignatedDealer_add_tier_"></a>

## Function `add_tier_`



<pre><code><b>fun</b> <a href="#0x1_DesignatedDealer_add_tier_">add_tier_</a>(dealer: &<b>mut</b> <a href="#0x1_DesignatedDealer_Dealer">DesignatedDealer::Dealer</a>, next_tier_upperbound: u64)
</code></pre>



<details>
<summary>Implementation</summary>


<pre><code><b>fun</b> <a href="#0x1_DesignatedDealer_add_tier_">add_tier_</a>(dealer: &<b>mut</b> <a href="#0x1_DesignatedDealer_Dealer">Dealer</a>, next_tier_upperbound: u64) {
    <b>let</b> tiers = &<b>mut</b> dealer.tiers;
    <b>let</b> number_of_tiers: u64 = <a href="Vector.md#0x1_Vector_length">Vector::length</a>(tiers);
    <b>assert</b>(number_of_tiers + 1 &lt;= MAX_NUM_TIERS, EINVALID_TIER_ADDITION);
    <b>if</b> (number_of_tiers &gt; 0) {
        <b>let</b> last_tier = *<a href="Vector.md#0x1_Vector_borrow">Vector::borrow</a>(tiers, number_of_tiers - 1);
        <b>assert</b>(last_tier &lt; next_tier_upperbound, EINVALID_TIER_START);
    };
    <a href="Vector.md#0x1_Vector_push_back">Vector::push_back</a>(tiers, next_tier_upperbound);
}
</code></pre>



</details>

<a name="0x1_DesignatedDealer_add_tier"></a>

## Function `add_tier`



<pre><code><b>public</b> <b>fun</b> <a href="#0x1_DesignatedDealer_add_tier">add_tier</a>(tc_account: &signer, dd_addr: address, tier_upperbound: u64)
</code></pre>



<details>
<summary>Implementation</summary>


<pre><code><b>public</b> <b>fun</b> <a href="#0x1_DesignatedDealer_add_tier">add_tier</a>(
    tc_account: &signer,
    dd_addr: address,
    tier_upperbound: u64
) <b>acquires</b> <a href="#0x1_DesignatedDealer_Dealer">Dealer</a> {
    <b>assert</b>(<a href="Roles.md#0x1_Roles_has_treasury_compliance_role">Roles::has_treasury_compliance_role</a>(tc_account), EACCOUNT_NOT_TREASURY_COMPLIANCE);
    <b>let</b> dealer = borrow_global_mut&lt;<a href="#0x1_DesignatedDealer_Dealer">Dealer</a>&gt;(dd_addr);
    <a href="#0x1_DesignatedDealer_add_tier_">add_tier_</a>(dealer, tier_upperbound)
}
</code></pre>



</details>

<a name="0x1_DesignatedDealer_update_tier_"></a>

## Function `update_tier_`



<pre><code><b>fun</b> <a href="#0x1_DesignatedDealer_update_tier_">update_tier_</a>(dealer: &<b>mut</b> <a href="#0x1_DesignatedDealer_Dealer">DesignatedDealer::Dealer</a>, tier_index: u64, new_upperbound: u64)
</code></pre>



<details>
<summary>Implementation</summary>


<pre><code><b>fun</b> <a href="#0x1_DesignatedDealer_update_tier_">update_tier_</a>(dealer: &<b>mut</b> <a href="#0x1_DesignatedDealer_Dealer">Dealer</a>, tier_index: u64, new_upperbound: u64) {
    <b>let</b> tiers = &<b>mut</b> dealer.tiers;
    <b>let</b> number_of_tiers = <a href="Vector.md#0x1_Vector_length">Vector::length</a>(tiers);
    <b>assert</b>(tier_index &lt; number_of_tiers, EINVALID_TIER_INDEX);
    // Make sure that this new start for the tier is consistent
    // with the tier above and below it.
    <b>let</b> tier = <a href="Vector.md#0x1_Vector_borrow">Vector::borrow</a>(tiers, tier_index);
    <b>if</b> (*tier == new_upperbound) <b>return</b>;
    <b>if</b> (*tier &lt; new_upperbound) {
        <b>let</b> next_tier_index = tier_index + 1;
        <b>if</b> (next_tier_index &lt; number_of_tiers) {
            <b>assert</b>(new_upperbound &lt; *<a href="Vector.md#0x1_Vector_borrow">Vector::borrow</a>(tiers, next_tier_index), EINVALID_TIER_START);
        };
    };
    <b>if</b> (*tier &gt; new_upperbound && tier_index &gt; 0) {
        <b>let</b> prev_tier_index = tier_index - 1;
        <b>assert</b>(new_upperbound &gt; *<a href="Vector.md#0x1_Vector_borrow">Vector::borrow</a>(tiers, prev_tier_index), EINVALID_TIER_START);
    };
    *<a href="Vector.md#0x1_Vector_borrow_mut">Vector::borrow_mut</a>(tiers, tier_index) = new_upperbound;
}
</code></pre>



</details>

<a name="0x1_DesignatedDealer_update_tier"></a>

## Function `update_tier`



<pre><code><b>public</b> <b>fun</b> <a href="#0x1_DesignatedDealer_update_tier">update_tier</a>(tc_account: &signer, dd_addr: address, tier_index: u64, new_upperbound: u64)
</code></pre>



<details>
<summary>Implementation</summary>


<pre><code><b>public</b> <b>fun</b> <a href="#0x1_DesignatedDealer_update_tier">update_tier</a>(
    tc_account: &signer,
    dd_addr: address,
    tier_index: u64,
    new_upperbound: u64
) <b>acquires</b> <a href="#0x1_DesignatedDealer_Dealer">Dealer</a> {
    <b>assert</b>(<a href="Roles.md#0x1_Roles_has_treasury_compliance_role">Roles::has_treasury_compliance_role</a>(tc_account), EACCOUNT_NOT_TREASURY_COMPLIANCE);
    <b>let</b> dealer = borrow_global_mut&lt;<a href="#0x1_DesignatedDealer_Dealer">Dealer</a>&gt;(dd_addr);
    <a href="#0x1_DesignatedDealer_update_tier_">update_tier_</a>(dealer, tier_index, new_upperbound)
}
</code></pre>



</details>

<a name="0x1_DesignatedDealer_tiered_mint_"></a>

## Function `tiered_mint_`



<pre><code><b>fun</b> <a href="#0x1_DesignatedDealer_tiered_mint_">tiered_mint_</a>(dealer: &<b>mut</b> <a href="#0x1_DesignatedDealer_Dealer">DesignatedDealer::Dealer</a>, amount: u64, tier_index: u64)
</code></pre>



<details>
<summary>Implementation</summary>


<pre><code><b>fun</b> <a href="#0x1_DesignatedDealer_tiered_mint_">tiered_mint_</a>(dealer: &<b>mut</b> <a href="#0x1_DesignatedDealer_Dealer">Dealer</a>, amount: u64, tier_index: u64) {
    <a href="#0x1_DesignatedDealer_reset_window">reset_window</a>(dealer);
    <b>let</b> cur_inflow = dealer.window_inflow;
    <b>let</b> new_inflow = cur_inflow + amount;
    <b>let</b> tiers = &<b>mut</b> dealer.tiers;
    <b>let</b> number_of_tiers = <a href="Vector.md#0x1_Vector_length">Vector::length</a>(tiers);
    <b>assert</b>(tier_index &lt; number_of_tiers, EINVALID_TIER_INDEX);
    <b>let</b> tier_upperbound: u64 = *<a href="Vector.md#0x1_Vector_borrow">Vector::borrow</a>(tiers, tier_index);
    <b>assert</b>(new_inflow &lt;= tier_upperbound, EINVALID_AMOUNT_FOR_TIER);
    dealer.window_inflow = new_inflow;
}
</code></pre>



</details>

<a name="0x1_DesignatedDealer_tiered_mint"></a>

## Function `tiered_mint`



<pre><code><b>public</b> <b>fun</b> <a href="#0x1_DesignatedDealer_tiered_mint">tiered_mint</a>&lt;CoinType&gt;(tc_account: &signer, amount: u64, dd_addr: address, tier_index: u64): <a href="Libra.md#0x1_Libra_Libra">Libra::Libra</a>&lt;CoinType&gt;
</code></pre>



<details>
<summary>Implementation</summary>


<pre><code><b>public</b> <b>fun</b> <a href="#0x1_DesignatedDealer_tiered_mint">tiered_mint</a>&lt;CoinType&gt;(
    tc_account: &signer,
    amount: u64,
    dd_addr: address,
    tier_index: u64,
): <a href="Libra.md#0x1_Libra">Libra</a>&lt;CoinType&gt; <b>acquires</b> <a href="#0x1_DesignatedDealer_Dealer">Dealer</a> {
    <b>assert</b>(<a href="Roles.md#0x1_Roles_has_treasury_compliance_role">Roles::has_treasury_compliance_role</a>(tc_account), EACCOUNT_NOT_TREASURY_COMPLIANCE);
    <b>assert</b>(amount &gt; 0, EINVALID_MINT_AMOUNT);
    <b>assert</b>(<a href="#0x1_DesignatedDealer_exists_at">exists_at</a>(dd_addr), ENOT_A_DD);

    <a href="#0x1_DesignatedDealer_tiered_mint_">tiered_mint_</a>(borrow_global_mut&lt;<a href="#0x1_DesignatedDealer_Dealer">Dealer</a>&gt;(dd_addr), amount, tier_index);

    // Send <a href="#0x1_DesignatedDealer_ReceivedMintEvent">ReceivedMintEvent</a>
    <a href="Event.md#0x1_Event_emit_event">Event::emit_event</a>&lt;<a href="#0x1_DesignatedDealer_ReceivedMintEvent">ReceivedMintEvent</a>&gt;(
        &<b>mut</b> borrow_global_mut&lt;<a href="#0x1_DesignatedDealer_Dealer">Dealer</a>&gt;(dd_addr).mint_event_handle,
        <a href="#0x1_DesignatedDealer_ReceivedMintEvent">ReceivedMintEvent</a> {
            destination_address: dd_addr,
            amount: amount,
        },
    );
    <a href="Libra.md#0x1_Libra_mint">Libra::mint</a>&lt;CoinType&gt;(tc_account, amount)
}
</code></pre>



</details>

<a name="0x1_DesignatedDealer_exists_at"></a>

## Function `exists_at`



<pre><code><b>public</b> <b>fun</b> <a href="#0x1_DesignatedDealer_exists_at">exists_at</a>(dd_addr: address): bool
</code></pre>



<details>
<summary>Implementation</summary>


<pre><code><b>public</b> <b>fun</b> <a href="#0x1_DesignatedDealer_exists_at">exists_at</a>(dd_addr: address): bool {
    exists&lt;<a href="#0x1_DesignatedDealer_Dealer">Dealer</a>&gt;(dd_addr)
}
</code></pre>



</details>

<a name="0x1_DesignatedDealer_reset_window"></a>

## Function `reset_window`



<pre><code><b>fun</b> <a href="#0x1_DesignatedDealer_reset_window">reset_window</a>(dealer: &<b>mut</b> <a href="#0x1_DesignatedDealer_Dealer">DesignatedDealer::Dealer</a>)
</code></pre>



<details>
<summary>Implementation</summary>


<pre><code><b>fun</b> <a href="#0x1_DesignatedDealer_reset_window">reset_window</a>(dealer: &<b>mut</b> <a href="#0x1_DesignatedDealer_Dealer">Dealer</a>) {
    <b>let</b> current_time = <a href="LibraTimestamp.md#0x1_LibraTimestamp_now_microseconds">LibraTimestamp::now_microseconds</a>();
    <b>if</b> (current_time &gt;= dealer.window_start + <a href="#0x1_DesignatedDealer_window_length">window_length</a>()) {
        dealer.window_start = current_time;
        dealer.window_inflow = 0;
    }
}
</code></pre>



</details>

<a name="0x1_DesignatedDealer_window_length"></a>

## Function `window_length`



<pre><code><b>fun</b> <a href="#0x1_DesignatedDealer_window_length">window_length</a>(): u64
</code></pre>



<details>
<summary>Implementation</summary>


<pre><code><b>fun</b> <a href="#0x1_DesignatedDealer_window_length">window_length</a>(): u64 {
    // number of microseconds in a day
    86400000000
}
</code></pre>



</details>

<a name="0x1_DesignatedDealer_Specification"></a>

## Specification


<a name="0x1_DesignatedDealer_Specification_add_tier"></a>

### Function `add_tier`


<pre><code><b>public</b> <b>fun</b> <a href="#0x1_DesignatedDealer_add_tier">add_tier</a>(tc_account: &signer, dd_addr: address, tier_upperbound: u64)
</code></pre>




<pre><code><b>ensures</b> len(<b>global</b>&lt;<a href="#0x1_DesignatedDealer_Dealer">Dealer</a>&gt;(dd_addr).tiers) == len(<b>old</b>(<b>global</b>&lt;<a href="#0x1_DesignatedDealer_Dealer">Dealer</a>&gt;(dd_addr)).tiers) + 1;
<b>ensures</b> <b>global</b>&lt;<a href="#0x1_DesignatedDealer_Dealer">Dealer</a>&gt;(dd_addr).tiers[len(<b>global</b>&lt;<a href="#0x1_DesignatedDealer_Dealer">Dealer</a>&gt;(dd_addr).tiers) - 1] == tier_upperbound;
</code></pre>



<a name="0x1_DesignatedDealer_Specification_update_tier"></a>

### Function `update_tier`


<pre><code><b>public</b> <b>fun</b> <a href="#0x1_DesignatedDealer_update_tier">update_tier</a>(tc_account: &signer, dd_addr: address, tier_index: u64, new_upperbound: u64)
</code></pre>




<pre><code><b>ensures</b> len(<b>global</b>&lt;<a href="#0x1_DesignatedDealer_Dealer">Dealer</a>&gt;(dd_addr).tiers) == len(<b>old</b>(<b>global</b>&lt;<a href="#0x1_DesignatedDealer_Dealer">Dealer</a>&gt;(dd_addr)).tiers);
<b>ensures</b> <b>global</b>&lt;<a href="#0x1_DesignatedDealer_Dealer">Dealer</a>&gt;(dd_addr).tiers[tier_index] == new_upperbound;
</code></pre>



<a name="0x1_DesignatedDealer_Specification_tiered_mint"></a>

### Function `tiered_mint`


<pre><code><b>public</b> <b>fun</b> <a href="#0x1_DesignatedDealer_tiered_mint">tiered_mint</a>&lt;CoinType&gt;(tc_account: &signer, amount: u64, dd_addr: address, tier_index: u64): <a href="Libra.md#0x1_Libra_Libra">Libra::Libra</a>&lt;CoinType&gt;
</code></pre>




<pre><code><b>ensures</b> {<b>let</b> dealer = <b>global</b>&lt;<a href="#0x1_DesignatedDealer_Dealer">Dealer</a>&gt;(dd_addr); <b>old</b>(dealer.window_start) &lt;= dealer.window_start};
<b>ensures</b> {<b>let</b> dealer = <b>global</b>&lt;<a href="#0x1_DesignatedDealer_Dealer">Dealer</a>&gt;(dd_addr);
        {<b>let</b> current_time = <a href="LibraTimestamp.md#0x1_LibraTimestamp_spec_now_microseconds">LibraTimestamp::spec_now_microseconds</a>();
            (dealer.window_start == current_time && dealer.window_inflow == amount) ||
            (<b>old</b>(dealer.window_start) == dealer.window_start && dealer.window_inflow == <b>old</b>(dealer.window_inflow) + amount)
        }};
<b>ensures</b> tier_index &lt; len(<b>old</b>(<b>global</b>&lt;<a href="#0x1_DesignatedDealer_Dealer">Dealer</a>&gt;(dd_addr)).tiers);
<b>ensures</b> <b>global</b>&lt;<a href="#0x1_DesignatedDealer_Dealer">Dealer</a>&gt;(dd_addr).window_inflow &lt;= <b>old</b>(<b>global</b>&lt;<a href="#0x1_DesignatedDealer_Dealer">Dealer</a>&gt;(dd_addr)).tiers[tier_index];
</code></pre>



<a name="0x1_DesignatedDealer_Specification_exists_at"></a>

### Function `exists_at`


<pre><code><b>public</b> <b>fun</b> <a href="#0x1_DesignatedDealer_exists_at">exists_at</a>(dd_addr: address): bool
</code></pre>




<pre><code>pragma verify = <b>true</b>;
<b>ensures</b> result == <a href="#0x1_DesignatedDealer_spec_exists_at">spec_exists_at</a>(dd_addr);
</code></pre>




<a name="0x1_DesignatedDealer_spec_exists_at"></a>


<pre><code><b>define</b> <a href="#0x1_DesignatedDealer_spec_exists_at">spec_exists_at</a>(addr: address): bool {
    exists&lt;<a href="#0x1_DesignatedDealer_Dealer">Dealer</a>&gt;(addr)
}
</code></pre>




<a name="0x1_DesignatedDealer_SpecSchema"></a>


<pre><code><b>schema</b> <a href="#0x1_DesignatedDealer_SpecSchema">SpecSchema</a> {
    <b>invariant</b> <b>module</b> forall x: address where exists&lt;<a href="#0x1_DesignatedDealer_Dealer">Dealer</a>&gt;(x): len(<b>global</b>&lt;<a href="#0x1_DesignatedDealer_Dealer">Dealer</a>&gt;(x).tiers) &lt;= <a href="#0x1_DesignatedDealer_SPEC_MAX_NUM_TIERS">SPEC_MAX_NUM_TIERS</a>();
    <b>invariant</b> <b>module</b> forall x: address where exists&lt;<a href="#0x1_DesignatedDealer_Dealer">Dealer</a>&gt;(x):
                     forall i: u64, j: u64 where 0 &lt;= i && i &lt; j && j &lt; len(<b>global</b>&lt;<a href="#0x1_DesignatedDealer_Dealer">Dealer</a>&gt;(x).tiers):
                        <b>global</b>&lt;<a href="#0x1_DesignatedDealer_Dealer">Dealer</a>&gt;(x).tiers[i] &lt; <b>global</b>&lt;<a href="#0x1_DesignatedDealer_Dealer">Dealer</a>&gt;(x).tiers[j];
}
</code></pre>




<pre><code>pragma verify = <b>false</b>;
<a name="0x1_DesignatedDealer_SPEC_MAX_NUM_TIERS"></a>
<b>define</b> <a href="#0x1_DesignatedDealer_SPEC_MAX_NUM_TIERS">SPEC_MAX_NUM_TIERS</a>(): u64 { 4 }
<a name="0x1_DesignatedDealer_spec_window_length"></a>
<b>define</b> <a href="#0x1_DesignatedDealer_spec_window_length">spec_window_length</a>(): u64 { 86400000000 }
<b>apply</b> <a href="#0x1_DesignatedDealer_SpecSchema">SpecSchema</a> <b>to</b> *, *&lt;CoinType&gt;;
</code></pre>
