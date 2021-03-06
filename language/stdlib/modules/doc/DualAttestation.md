
<a name="0x1_DualAttestation"></a>

# Module `0x1::DualAttestation`

### Table of Contents

-  [Resource `Credential`](#0x1_DualAttestation_Credential)
-  [Function `publish_credential`](#0x1_DualAttestation_publish_credential)
-  [Function `rotate_base_url`](#0x1_DualAttestation_rotate_base_url)
-  [Function `rotate_compliance_public_key`](#0x1_DualAttestation_rotate_compliance_public_key)
-  [Function `human_name`](#0x1_DualAttestation_human_name)
-  [Function `base_url`](#0x1_DualAttestation_base_url)
-  [Function `compliance_public_key`](#0x1_DualAttestation_compliance_public_key)
-  [Function `expiration_date`](#0x1_DualAttestation_expiration_date)
-  [Function `recertify`](#0x1_DualAttestation_recertify)
-  [Function `decertify`](#0x1_DualAttestation_decertify)
-  [Function `credential_address`](#0x1_DualAttestation_credential_address)
-  [Function `dual_attestation_required`](#0x1_DualAttestation_dual_attestation_required)
-  [Function `dual_attestation_message`](#0x1_DualAttestation_dual_attestation_message)
-  [Function `assert_signature_is_valid`](#0x1_DualAttestation_assert_signature_is_valid)
-  [Function `assert_payment_ok`](#0x1_DualAttestation_assert_payment_ok)
-  [Specification](#0x1_DualAttestation_Specification)
    -  [Function `rotate_base_url`](#0x1_DualAttestation_Specification_rotate_base_url)
    -  [Function `rotate_compliance_public_key`](#0x1_DualAttestation_Specification_rotate_compliance_public_key)
    -  [Function `recertify`](#0x1_DualAttestation_Specification_recertify)
    -  [Function `decertify`](#0x1_DualAttestation_Specification_decertify)
    -  [Function `credential_address`](#0x1_DualAttestation_Specification_credential_address)
    -  [Function `dual_attestation_required`](#0x1_DualAttestation_Specification_dual_attestation_required)
    -  [Function `dual_attestation_message`](#0x1_DualAttestation_Specification_dual_attestation_message)
    -  [Function `assert_signature_is_valid`](#0x1_DualAttestation_Specification_assert_signature_is_valid)



<a name="0x1_DualAttestation_Credential"></a>

## Resource `Credential`

This resource holds an entity's globally unique name and all of the metadata it needs to
participate in off-chain protocols.


<pre><code><b>resource</b> <b>struct</b> <a href="#0x1_DualAttestation_Credential">Credential</a>
</code></pre>



<details>
<summary>Fields</summary>


<dl>
<dt>

<code>human_name: vector&lt;u8&gt;</code>
</dt>
<dd>
 The human readable name of this entity. Immutable.
</dd>
<dt>

<code>base_url: vector&lt;u8&gt;</code>
</dt>
<dd>
 The base_url holds the URL to be used for off-chain communication. This contains the
 entire URL (e.g. https://...). Mutable.
</dd>
<dt>

<code>compliance_public_key: vector&lt;u8&gt;</code>
</dt>
<dd>
 32 byte Ed25519 public key whose counterpart must be used to sign
 (1) the payment metadata for on-chain transactions that require dual attestation (e.g.,
     transactions subject to the travel rule)
 (2) information exchanged in the off-chain protocols (e.g., KYC info in the travel rule
     protocol)
 Note that this is different than
<code>authentication_key</code> used in LibraAccount, which is
 a hash of a public key + signature scheme identifier, not a public key. Mutable.
</dd>
<dt>

<code>expiration_date: u64</code>
</dt>
<dd>
 Expiration date in microseconds from unix epoch. For V1, it is always set to
 U64_MAX. Mutable, but only by LibraRoot.
</dd>
</dl>


</details>

<a name="0x1_DualAttestation_publish_credential"></a>

## Function `publish_credential`



<pre><code><b>public</b> <b>fun</b> <a href="#0x1_DualAttestation_publish_credential">publish_credential</a>(created: &signer, creator: &signer, human_name: vector&lt;u8&gt;, base_url: vector&lt;u8&gt;, compliance_public_key: vector&lt;u8&gt;)
</code></pre>



<details>
<summary>Implementation</summary>


<pre><code><b>public</b> <b>fun</b> <a href="#0x1_DualAttestation_publish_credential">publish_credential</a>(
    created: &signer,
    creator: &signer,
    human_name: vector&lt;u8&gt;,
    base_url: vector&lt;u8&gt;,
    compliance_public_key: vector&lt;u8&gt;,
) {
    <b>assert</b>(
        <a href="Roles.md#0x1_Roles_has_parent_VASP_role">Roles::has_parent_VASP_role</a>(created) || <a href="Roles.md#0x1_Roles_has_designated_dealer_role">Roles::has_designated_dealer_role</a>(created),
        ENOT_PARENT_VASP_OR_DD
    );
    <b>assert</b>(
        <a href="Roles.md#0x1_Roles_has_libra_root_role">Roles::has_libra_root_role</a>(creator) || <a href="Roles.md#0x1_Roles_has_treasury_compliance_role">Roles::has_treasury_compliance_role</a>(creator),
        ENOT_LIBRA_ROOT_OR_TREASURY_COMPLIANCE
    );
    <b>assert</b>(<a href="Signature.md#0x1_Signature_ed25519_validate_pubkey">Signature::ed25519_validate_pubkey</a>(<b>copy</b> compliance_public_key), EINVALID_PUBLIC_KEY);
    move_to(created, <a href="#0x1_DualAttestation_Credential">Credential</a> {
        human_name,
        base_url,
        compliance_public_key,
        // For testnet and V1, so it should never expire. So set <b>to</b> u64::MAX
        expiration_date: U64_MAX,
    })
}
</code></pre>



</details>

<a name="0x1_DualAttestation_rotate_base_url"></a>

## Function `rotate_base_url`

Rotate the base URL for
<code>account</code> to
<code>new_url</code>


<pre><code><b>public</b> <b>fun</b> <a href="#0x1_DualAttestation_rotate_base_url">rotate_base_url</a>(account: &signer, new_url: vector&lt;u8&gt;)
</code></pre>



<details>
<summary>Implementation</summary>


<pre><code><b>public</b> <b>fun</b> <a href="#0x1_DualAttestation_rotate_base_url">rotate_base_url</a>(account: &signer, new_url: vector&lt;u8&gt;) <b>acquires</b> <a href="#0x1_DualAttestation_Credential">Credential</a> {
    borrow_global_mut&lt;<a href="#0x1_DualAttestation_Credential">Credential</a>&gt;(<a href="Signer.md#0x1_Signer_address_of">Signer::address_of</a>(account)).base_url = new_url
}
</code></pre>



</details>

<a name="0x1_DualAttestation_rotate_compliance_public_key"></a>

## Function `rotate_compliance_public_key`

Rotate the compliance public key for
<code>account</code> to
<code>new_key</code>.


<pre><code><b>public</b> <b>fun</b> <a href="#0x1_DualAttestation_rotate_compliance_public_key">rotate_compliance_public_key</a>(account: &signer, new_key: vector&lt;u8&gt;)
</code></pre>



<details>
<summary>Implementation</summary>


<pre><code><b>public</b> <b>fun</b> <a href="#0x1_DualAttestation_rotate_compliance_public_key">rotate_compliance_public_key</a>(
    account: &signer,
    new_key: vector&lt;u8&gt;,
) <b>acquires</b> <a href="#0x1_DualAttestation_Credential">Credential</a> {
    <b>assert</b>(<a href="Signature.md#0x1_Signature_ed25519_validate_pubkey">Signature::ed25519_validate_pubkey</a>(<b>copy</b> new_key), EINVALID_PUBLIC_KEY);
    <b>let</b> parent_addr = <a href="Signer.md#0x1_Signer_address_of">Signer::address_of</a>(account);
    borrow_global_mut&lt;<a href="#0x1_DualAttestation_Credential">Credential</a>&gt;(parent_addr).compliance_public_key = new_key
}
</code></pre>



</details>

<a name="0x1_DualAttestation_human_name"></a>

## Function `human_name`

Return the human-readable name for the VASP account.
Aborts if
<code>addr</code> does not have a
<code><a href="#0x1_DualAttestation_Credential">Credential</a></code> resource.


<pre><code><b>public</b> <b>fun</b> <a href="#0x1_DualAttestation_human_name">human_name</a>(addr: address): vector&lt;u8&gt;
</code></pre>



<details>
<summary>Implementation</summary>


<pre><code><b>public</b> <b>fun</b> <a href="#0x1_DualAttestation_human_name">human_name</a>(addr: address): vector&lt;u8&gt; <b>acquires</b> <a href="#0x1_DualAttestation_Credential">Credential</a> {
    *&borrow_global&lt;<a href="#0x1_DualAttestation_Credential">Credential</a>&gt;(addr).human_name
}
</code></pre>



</details>

<a name="0x1_DualAttestation_base_url"></a>

## Function `base_url`

Return the base URL for
<code>addr</code>.
Aborts if
<code>addr</code> does not have a
<code><a href="#0x1_DualAttestation_Credential">Credential</a></code> resource.


<pre><code><b>public</b> <b>fun</b> <a href="#0x1_DualAttestation_base_url">base_url</a>(addr: address): vector&lt;u8&gt;
</code></pre>



<details>
<summary>Implementation</summary>


<pre><code><b>public</b> <b>fun</b> <a href="#0x1_DualAttestation_base_url">base_url</a>(addr: address): vector&lt;u8&gt; <b>acquires</b> <a href="#0x1_DualAttestation_Credential">Credential</a> {
    *&borrow_global&lt;<a href="#0x1_DualAttestation_Credential">Credential</a>&gt;(addr).base_url
}
</code></pre>



</details>

<a name="0x1_DualAttestation_compliance_public_key"></a>

## Function `compliance_public_key`

Return the compliance public key for
<code>addr</code>.
Aborts if
<code>addr</code> does not have a
<code><a href="#0x1_DualAttestation_Credential">Credential</a></code> resource.


<pre><code><b>public</b> <b>fun</b> <a href="#0x1_DualAttestation_compliance_public_key">compliance_public_key</a>(addr: address): vector&lt;u8&gt;
</code></pre>



<details>
<summary>Implementation</summary>


<pre><code><b>public</b> <b>fun</b> <a href="#0x1_DualAttestation_compliance_public_key">compliance_public_key</a>(addr: address): vector&lt;u8&gt; <b>acquires</b> <a href="#0x1_DualAttestation_Credential">Credential</a> {
    *&borrow_global&lt;<a href="#0x1_DualAttestation_Credential">Credential</a>&gt;(addr).compliance_public_key
}
</code></pre>



</details>

<a name="0x1_DualAttestation_expiration_date"></a>

## Function `expiration_date`

Return the expiration date
<code>addr
Aborts <b>if</b> </code>addr
<code> does not have a </code>Credential` resource.


<pre><code><b>public</b> <b>fun</b> <a href="#0x1_DualAttestation_expiration_date">expiration_date</a>(addr: address): u64
</code></pre>



<details>
<summary>Implementation</summary>


<pre><code><b>public</b> <b>fun</b> <a href="#0x1_DualAttestation_expiration_date">expiration_date</a>(addr: address): u64  <b>acquires</b> <a href="#0x1_DualAttestation_Credential">Credential</a> {
    *&borrow_global&lt;<a href="#0x1_DualAttestation_Credential">Credential</a>&gt;(addr).expiration_date
}
</code></pre>



</details>

<a name="0x1_DualAttestation_recertify"></a>

## Function `recertify`

Renew's
<code>credential</code>'s certificate


<pre><code><b>public</b> <b>fun</b> <a href="#0x1_DualAttestation_recertify">recertify</a>(credential: &<b>mut</b> <a href="#0x1_DualAttestation_Credential">DualAttestation::Credential</a>)
</code></pre>



<details>
<summary>Implementation</summary>


<pre><code><b>public</b> <b>fun</b> <a href="#0x1_DualAttestation_recertify">recertify</a>(credential: &<b>mut</b> <a href="#0x1_DualAttestation_Credential">Credential</a>) {
    credential.expiration_date = <a href="LibraTimestamp.md#0x1_LibraTimestamp_now_microseconds">LibraTimestamp::now_microseconds</a>() + ONE_YEAR;
}
</code></pre>



</details>

<a name="0x1_DualAttestation_decertify"></a>

## Function `decertify`

Non-destructively decertify
<code>credential</code>. Can be recertified later on via
<code>recertify</code>.


<pre><code><b>public</b> <b>fun</b> <a href="#0x1_DualAttestation_decertify">decertify</a>(credential: &<b>mut</b> <a href="#0x1_DualAttestation_Credential">DualAttestation::Credential</a>)
</code></pre>



<details>
<summary>Implementation</summary>


<pre><code><b>public</b> <b>fun</b> <a href="#0x1_DualAttestation_decertify">decertify</a>(credential: &<b>mut</b> <a href="#0x1_DualAttestation_Credential">Credential</a>) {
    // Expire the parent credential.
    credential.expiration_date = 0;
}
</code></pre>



</details>

<a name="0x1_DualAttestation_credential_address"></a>

## Function `credential_address`

Return the address where the credentials for
<code>addr</code> are stored


<pre><code><b>fun</b> <a href="#0x1_DualAttestation_credential_address">credential_address</a>(addr: address): address
</code></pre>



<details>
<summary>Implementation</summary>


<pre><code><b>fun</b> <a href="#0x1_DualAttestation_credential_address">credential_address</a>(addr: address): address {
    <b>if</b> (<a href="VASP.md#0x1_VASP_is_child">VASP::is_child</a>(addr)) <a href="VASP.md#0x1_VASP_parent_address">VASP::parent_address</a>(addr) <b>else</b> addr
}
</code></pre>



</details>

<a name="0x1_DualAttestation_dual_attestation_required"></a>

## Function `dual_attestation_required`

Helper which returns true if dual attestion is required for a deposit.


<pre><code><b>fun</b> <a href="#0x1_DualAttestation_dual_attestation_required">dual_attestation_required</a>&lt;Token&gt;(payer: address, payee: address, deposit_value: u64): bool
</code></pre>



<details>
<summary>Implementation</summary>


<pre><code><b>fun</b> <a href="#0x1_DualAttestation_dual_attestation_required">dual_attestation_required</a>&lt;Token&gt;(
    payer: address, payee: address, deposit_value: u64
): bool {
    // travel rule applies for payments over a threshold
    <b>let</b> travel_rule_limit_microlibra = <a href="DualAttestationLimit.md#0x1_DualAttestationLimit_get_cur_microlibra_limit">DualAttestationLimit::get_cur_microlibra_limit</a>();
    <b>let</b> approx_lbr_microlibra_value = <a href="Libra.md#0x1_Libra_approx_lbr_for_value">Libra::approx_lbr_for_value</a>&lt;Token&gt;(deposit_value);
    <b>let</b> above_threshold = approx_lbr_microlibra_value &gt;= travel_rule_limit_microlibra;
    <b>if</b> (!above_threshold) {
        <b>return</b> <b>false</b>
    };
    // self-deposits never require dual attestation
    <b>if</b> (payer == payee) {
        <b>return</b> <b>false</b>
    };
    // dual attestation is required <b>if</b> the amount is above the threshold AND between distinct
    // entities. E.g.:
    // (1) inter-<a href="VASP.md#0x1_VASP">VASP</a>
    // (2) inter-DD
    // (3) <a href="VASP.md#0x1_VASP">VASP</a> -&gt; DD
    // (4) DD -&gt; <a href="VASP.md#0x1_VASP">VASP</a>
    // We <b>assume</b> that any DD &lt;-&gt; DD payment is inter-DD because each DD has a single account
    <b>let</b> is_payer_vasp = <a href="VASP.md#0x1_VASP_is_vasp">VASP::is_vasp</a>(payer);
    <b>let</b> is_payee_vasp = <a href="VASP.md#0x1_VASP_is_vasp">VASP::is_vasp</a>(payee);
    <b>let</b> is_payer_dd = <a href="DesignatedDealer.md#0x1_DesignatedDealer_exists_at">DesignatedDealer::exists_at</a>(payer);
    <b>let</b> is_payee_dd = <a href="DesignatedDealer.md#0x1_DesignatedDealer_exists_at">DesignatedDealer::exists_at</a>(payee);
    <b>let</b> is_inter_vasp =
        is_payer_vasp && is_payee_vasp &&
        <a href="VASP.md#0x1_VASP_parent_address">VASP::parent_address</a>(payer) != <a href="VASP.md#0x1_VASP_parent_address">VASP::parent_address</a>(payee);
    is_inter_vasp || // (1) inter-<a href="VASP.md#0x1_VASP">VASP</a>
        (is_payer_dd && is_payee_dd) || // (2) inter-DD
        (is_payer_vasp && is_payee_dd) || // (3) <a href="VASP.md#0x1_VASP">VASP</a> -&gt; DD
        (is_payer_dd && is_payee_vasp) // (4) DD -&gt; <a href="VASP.md#0x1_VASP">VASP</a>
}
</code></pre>



</details>

<a name="0x1_DualAttestation_dual_attestation_message"></a>

## Function `dual_attestation_message`

Helper to construct a message for dual attestation.
Message is
<code>metadata</code> |
<code>payer</code> |
<code>amount</code> |
<code>DOMAIN_SEPARATOR</code>.


<pre><code><b>fun</b> <a href="#0x1_DualAttestation_dual_attestation_message">dual_attestation_message</a>(payer: address, metadata: vector&lt;u8&gt;, deposit_value: u64): vector&lt;u8&gt;
</code></pre>



<details>
<summary>Implementation</summary>


<pre><code><b>fun</b> <a href="#0x1_DualAttestation_dual_attestation_message">dual_attestation_message</a>(
    payer: address, metadata: vector&lt;u8&gt;, deposit_value: u64
): vector&lt;u8&gt; {
    <b>let</b> message = metadata;
    <a href="Vector.md#0x1_Vector_append">Vector::append</a>(&<b>mut</b> message, <a href="LCS.md#0x1_LCS_to_bytes">LCS::to_bytes</a>(&payer));
    <a href="Vector.md#0x1_Vector_append">Vector::append</a>(&<b>mut</b> message, <a href="LCS.md#0x1_LCS_to_bytes">LCS::to_bytes</a>(&deposit_value));
    <a href="Vector.md#0x1_Vector_append">Vector::append</a>(&<b>mut</b> message, DOMAIN_SEPARATOR);
    message
}
</code></pre>



</details>

<a name="0x1_DualAttestation_assert_signature_is_valid"></a>

## Function `assert_signature_is_valid`

Helper function to check validity of a signature when dual attestion is required.


<pre><code><b>fun</b> <a href="#0x1_DualAttestation_assert_signature_is_valid">assert_signature_is_valid</a>(payer: address, payee: address, metadata_signature: vector&lt;u8&gt;, metadata: vector&lt;u8&gt;, deposit_value: u64)
</code></pre>



<details>
<summary>Implementation</summary>


<pre><code><b>fun</b> <a href="#0x1_DualAttestation_assert_signature_is_valid">assert_signature_is_valid</a>(
    payer: address,
    payee: address,
    metadata_signature: vector&lt;u8&gt;,
    metadata: vector&lt;u8&gt;,
    deposit_value: u64
) <b>acquires</b> <a href="#0x1_DualAttestation_Credential">Credential</a> {
    // sanity check of signature validity
    <b>assert</b>(<a href="Vector.md#0x1_Vector_length">Vector::length</a>(&metadata_signature) == 64, EMALFORMED_METADATA_SIGNATURE);
    // cryptographic check of signature validity
    <b>let</b> message = <a href="#0x1_DualAttestation_dual_attestation_message">dual_attestation_message</a>(payer, metadata, deposit_value);
    <b>assert</b>(
        <a href="Signature.md#0x1_Signature_ed25519_verify">Signature::ed25519_verify</a>(
            metadata_signature,
            <a href="#0x1_DualAttestation_compliance_public_key">compliance_public_key</a>(<a href="#0x1_DualAttestation_credential_address">credential_address</a>(payee)),
            message
        ),
        EINVALID_METADATA_SIGNATURE
    );
}
</code></pre>



</details>

<a name="0x1_DualAttestation_assert_payment_ok"></a>

## Function `assert_payment_ok`

Public API for checking whether a payment of
<code>value</code> coins of type
<code>Currency</code>
from
<code>payer</code> to
<code>payee</code> has a valid dual attestation. This returns without aborting if
(1) dual attestation is not required for this payment, or
(2) dual attestation is required, and
<code>metadata_signature</code> can be verified on the message
<code>metadata</code> |
<code>payer</code> |
<code>value</code> |
<code>DOMAIN_SEPARATOR</code> using the
<code>compliance_public_key</code>
published in
<code>payee</code>'s
<code><a href="#0x1_DualAttestation_Credential">Credential</a></code> resource
It aborts with an appropriate error code if dual attestation is required, but one or more of
the conditions in (2) is not met.


<pre><code><b>public</b> <b>fun</b> <a href="#0x1_DualAttestation_assert_payment_ok">assert_payment_ok</a>&lt;Currency&gt;(payer: address, payee: address, value: u64, metadata: vector&lt;u8&gt;, metadata_signature: vector&lt;u8&gt;)
</code></pre>



<details>
<summary>Implementation</summary>


<pre><code><b>public</b> <b>fun</b> <a href="#0x1_DualAttestation_assert_payment_ok">assert_payment_ok</a>&lt;Currency&gt;(
    payer: address,
    payee: address,
    value: u64,
    metadata: vector&lt;u8&gt;,
    metadata_signature: vector&lt;u8&gt;
) <b>acquires</b> <a href="#0x1_DualAttestation_Credential">Credential</a> {
    <b>if</b> (<a href="#0x1_DualAttestation_dual_attestation_required">dual_attestation_required</a>&lt;Currency&gt;(payer, payee, value)) {
      <a href="#0x1_DualAttestation_assert_signature_is_valid">assert_signature_is_valid</a>(payer, payee, metadata_signature, metadata, value)
    }
}
</code></pre>



</details>

<a name="0x1_DualAttestation_Specification"></a>

## Specification


<a name="0x1_DualAttestation_Specification_rotate_base_url"></a>

### Function `rotate_base_url`


<pre><code><b>public</b> <b>fun</b> <a href="#0x1_DualAttestation_rotate_base_url">rotate_base_url</a>(account: &signer, new_url: vector&lt;u8&gt;)
</code></pre>




<pre><code><b>aborts_if</b> !exists&lt;<a href="#0x1_DualAttestation_Credential">Credential</a>&gt;(<a href="Signer.md#0x1_Signer_spec_address_of">Signer::spec_address_of</a>(account));
<b>ensures</b>
    <b>global</b>&lt;<a href="#0x1_DualAttestation_Credential">Credential</a>&gt;(<a href="Signer.md#0x1_Signer_spec_address_of">Signer::spec_address_of</a>(account)).base_url == new_url;
</code></pre>



<a name="0x1_DualAttestation_Specification_rotate_compliance_public_key"></a>

### Function `rotate_compliance_public_key`


<pre><code><b>public</b> <b>fun</b> <a href="#0x1_DualAttestation_rotate_compliance_public_key">rotate_compliance_public_key</a>(account: &signer, new_key: vector&lt;u8&gt;)
</code></pre>




<pre><code><b>aborts_if</b> !exists&lt;<a href="#0x1_DualAttestation_Credential">Credential</a>&gt;(<a href="Signer.md#0x1_Signer_spec_address_of">Signer::spec_address_of</a>(account));
<b>aborts_if</b> !<a href="Signature.md#0x1_Signature_spec_ed25519_validate_pubkey">Signature::spec_ed25519_validate_pubkey</a>(new_key);
<b>ensures</b> <b>global</b>&lt;<a href="#0x1_DualAttestation_Credential">Credential</a>&gt;(<a href="Signer.md#0x1_Signer_spec_address_of">Signer::spec_address_of</a>(account)).compliance_public_key
     == new_key;
</code></pre>



Spec version of
<code><a href="#0x1_DualAttestation_compliance_public_key">Self::compliance_public_key</a></code>.


<a name="0x1_DualAttestation_spec_compliance_public_key"></a>


<pre><code><b>define</b> <a href="#0x1_DualAttestation_spec_compliance_public_key">spec_compliance_public_key</a>(addr: address): vector&lt;u8&gt; {
    <b>global</b>&lt;<a href="#0x1_DualAttestation_Credential">Credential</a>&gt;(addr).compliance_public_key
}
</code></pre>



<a name="0x1_DualAttestation_Specification_recertify"></a>

### Function `recertify`


<pre><code><b>public</b> <b>fun</b> <a href="#0x1_DualAttestation_recertify">recertify</a>(credential: &<b>mut</b> <a href="#0x1_DualAttestation_Credential">DualAttestation::Credential</a>)
</code></pre>




<pre><code><b>aborts_if</b> !<a href="LibraTimestamp.md#0x1_LibraTimestamp_root_ctm_initialized">LibraTimestamp::root_ctm_initialized</a>();
<b>aborts_if</b> <a href="LibraTimestamp.md#0x1_LibraTimestamp_spec_now_microseconds">LibraTimestamp::spec_now_microseconds</a>() + 31540000000000 &gt; max_u64();
<b>ensures</b> credential.expiration_date
     == <a href="LibraTimestamp.md#0x1_LibraTimestamp_spec_now_microseconds">LibraTimestamp::spec_now_microseconds</a>() + 31540000000000;
</code></pre>



<a name="0x1_DualAttestation_Specification_decertify"></a>

### Function `decertify`


<pre><code><b>public</b> <b>fun</b> <a href="#0x1_DualAttestation_decertify">decertify</a>(credential: &<b>mut</b> <a href="#0x1_DualAttestation_Credential">DualAttestation::Credential</a>)
</code></pre>




<pre><code><b>aborts_if</b> <b>false</b>;
<b>ensures</b> credential.expiration_date == 0;
</code></pre>



<a name="0x1_DualAttestation_Specification_credential_address"></a>

### Function `credential_address`


<pre><code><b>fun</b> <a href="#0x1_DualAttestation_credential_address">credential_address</a>(addr: address): address
</code></pre>




<pre><code><b>aborts_if</b> <b>false</b>;
<b>ensures</b> result == <a href="#0x1_DualAttestation_spec_credential_address">spec_credential_address</a>(addr);
</code></pre>




<a name="0x1_DualAttestation_spec_credential_address"></a>


<pre><code><b>define</b> <a href="#0x1_DualAttestation_spec_credential_address">spec_credential_address</a>(addr: address): address {
    <b>if</b> (<a href="VASP.md#0x1_VASP_spec_is_child_vasp">VASP::spec_is_child_vasp</a>(addr)) {
        <a href="VASP.md#0x1_VASP_spec_parent_address">VASP::spec_parent_address</a>(addr)
    } <b>else</b> {
        addr
    }
}
</code></pre>



<a name="0x1_DualAttestation_Specification_dual_attestation_required"></a>

### Function `dual_attestation_required`


<pre><code><b>fun</b> <a href="#0x1_DualAttestation_dual_attestation_required">dual_attestation_required</a>&lt;Token&gt;(payer: address, payee: address, deposit_value: u64): bool
</code></pre>




<pre><code>pragma opaque = <b>true</b>;
<b>include</b> <a href="#0x1_DualAttestation_TravelRuleAppliesAbortsIf">TravelRuleAppliesAbortsIf</a>&lt;Token&gt;;
<b>ensures</b> result == <a href="#0x1_DualAttestation_spec_dual_attestation_required">spec_dual_attestation_required</a>&lt;Token&gt;(payer, payee, deposit_value);
</code></pre>




<a name="0x1_DualAttestation_TravelRuleAppliesAbortsIf"></a>


<pre><code><b>schema</b> <a href="#0x1_DualAttestation_TravelRuleAppliesAbortsIf">TravelRuleAppliesAbortsIf</a>&lt;Token&gt; {
    <b>aborts_if</b> !<a href="Libra.md#0x1_Libra_spec_is_currency">Libra::spec_is_currency</a>&lt;Token&gt;();
    <b>aborts_if</b> !<a href="DualAttestationLimit.md#0x1_DualAttestationLimit_spec_is_published">DualAttestationLimit::spec_is_published</a>();
}
</code></pre>




<a name="0x1_DualAttestation_spec_is_inter_vasp"></a>


<pre><code><b>define</b> <a href="#0x1_DualAttestation_spec_is_inter_vasp">spec_is_inter_vasp</a>(payer: address, payee: address): bool {
    <a href="VASP.md#0x1_VASP_spec_is_vasp">VASP::spec_is_vasp</a>(payer) && <a href="VASP.md#0x1_VASP_spec_is_vasp">VASP::spec_is_vasp</a>(payee)
        && <a href="VASP.md#0x1_VASP_spec_parent_address">VASP::spec_parent_address</a>(payer) != <a href="VASP.md#0x1_VASP_spec_parent_address">VASP::spec_parent_address</a>(payee)
}
<a name="0x1_DualAttestation_spec_is_inter_dd"></a>
<b>define</b> <a href="#0x1_DualAttestation_spec_is_inter_dd">spec_is_inter_dd</a>(payer: address, payee: address): bool {
    <a href="DesignatedDealer.md#0x1_DesignatedDealer_spec_exists_at">DesignatedDealer::spec_exists_at</a>(payer) && <a href="DesignatedDealer.md#0x1_DesignatedDealer_spec_exists_at">DesignatedDealer::spec_exists_at</a>(payee)
}
<a name="0x1_DualAttestation_spec_is_vasp_to_dd"></a>
<b>define</b> <a href="#0x1_DualAttestation_spec_is_vasp_to_dd">spec_is_vasp_to_dd</a>(payer: address, payee: address): bool {
    <a href="VASP.md#0x1_VASP_spec_is_vasp">VASP::spec_is_vasp</a>(payer) && <a href="DesignatedDealer.md#0x1_DesignatedDealer_spec_exists_at">DesignatedDealer::spec_exists_at</a>(payee)
}
<a name="0x1_DualAttestation_spec_is_dd_to_vasp"></a>
<b>define</b> <a href="#0x1_DualAttestation_spec_is_dd_to_vasp">spec_is_dd_to_vasp</a>(payer: address, payee: address): bool {
    <a href="DesignatedDealer.md#0x1_DesignatedDealer_spec_exists_at">DesignatedDealer::spec_exists_at</a>(payer) && <a href="VASP.md#0x1_VASP_spec_is_vasp">VASP::spec_is_vasp</a>(payee)
}
</code></pre>


Helper functions which simulates
<code><a href="#0x1_DualAttestation_dual_attestation_required">Self::dual_attestation_required</a></code>.


<a name="0x1_DualAttestation_spec_dual_attestation_required"></a>


<pre><code><b>define</b> <a href="#0x1_DualAttestation_spec_dual_attestation_required">spec_dual_attestation_required</a>&lt;Token&gt;(
    payer: address, payee: address, deposit_value: u64
): bool {
    <a href="Libra.md#0x1_Libra_spec_approx_lbr_for_value">Libra::spec_approx_lbr_for_value</a>&lt;Token&gt;(deposit_value)
            &gt;= <a href="DualAttestationLimit.md#0x1_DualAttestationLimit_spec_get_cur_microlibra_limit">DualAttestationLimit::spec_get_cur_microlibra_limit</a>() &&
    payer != payee &&
    (<a href="#0x1_DualAttestation_spec_is_inter_vasp">spec_is_inter_vasp</a>(payer, payee) ||
     <a href="#0x1_DualAttestation_spec_is_inter_dd">spec_is_inter_dd</a>(payer, payee) ||
     <a href="#0x1_DualAttestation_spec_is_vasp_to_dd">spec_is_vasp_to_dd</a>(payer, payee) ||
     <a href="#0x1_DualAttestation_spec_is_dd_to_vasp">spec_is_dd_to_vasp</a>(payer, payee))
}
</code></pre>



<a name="0x1_DualAttestation_Specification_dual_attestation_message"></a>

### Function `dual_attestation_message`


<pre><code><b>fun</b> <a href="#0x1_DualAttestation_dual_attestation_message">dual_attestation_message</a>(payer: address, metadata: vector&lt;u8&gt;, deposit_value: u64): vector&lt;u8&gt;
</code></pre>



Abstract from construction of message for the prover. Concatenation of results from
<code><a href="LCS.md#0x1_LCS_to_bytes">LCS::to_bytes</a></code>
are difficult to reason about, so we avoid doing it. This is possible because the actual value of this
message is not important for the verification problem, as long as the prover considers both
messages which fail verification and which do not.


<pre><code>pragma opaque = <b>true</b>, verify = <b>false</b>;
<b>ensures</b> result == <a href="#0x1_DualAttestation_spec_dual_attestation_message">spec_dual_attestation_message</a>(payer, metadata, deposit_value);
</code></pre>



Uninterpreted function for
<code><a href="#0x1_DualAttestation_dual_attestation_message">Self::dual_attestation_message</a></code>.


<a name="0x1_DualAttestation_spec_dual_attestation_message"></a>


<pre><code><b>define</b> <a href="#0x1_DualAttestation_spec_dual_attestation_message">spec_dual_attestation_message</a>(payer: address, metadata: vector&lt;u8&gt;, deposit_value: u64): vector&lt;u8&gt;;
</code></pre>



<a name="0x1_DualAttestation_Specification_assert_signature_is_valid"></a>

### Function `assert_signature_is_valid`


<pre><code><b>fun</b> <a href="#0x1_DualAttestation_assert_signature_is_valid">assert_signature_is_valid</a>(payer: address, payee: address, metadata_signature: vector&lt;u8&gt;, metadata: vector&lt;u8&gt;, deposit_value: u64)
</code></pre>




<pre><code>pragma opaque = <b>true</b>;
<b>aborts_if</b> !exists&lt;<a href="#0x1_DualAttestation_Credential">Credential</a>&gt;(<a href="#0x1_DualAttestation_spec_credential_address">spec_credential_address</a>(payee));
<b>aborts_if</b> !<a href="#0x1_DualAttestation_signature_is_valid">signature_is_valid</a>(payer, payee, metadata_signature, metadata, deposit_value);
</code></pre>



Returns true if signature is valid.


<a name="0x1_DualAttestation_signature_is_valid"></a>


<pre><code><b>define</b> <a href="#0x1_DualAttestation_signature_is_valid">signature_is_valid</a>(
    payer: address,
    payee: address,
    metadata_signature: vector&lt;u8&gt;,
    metadata: vector&lt;u8&gt;,
    deposit_value: u64
): bool {
    len(metadata_signature) == 64
        && <a href="Signature.md#0x1_Signature_spec_ed25519_verify">Signature::spec_ed25519_verify</a>(
                metadata_signature,
                <a href="#0x1_DualAttestation_spec_compliance_public_key">spec_compliance_public_key</a>(<a href="#0x1_DualAttestation_spec_credential_address">spec_credential_address</a>(payee)),
                <a href="#0x1_DualAttestation_spec_dual_attestation_message">spec_dual_attestation_message</a>(payer, metadata, deposit_value)
           )
}
</code></pre>




<pre><code>pragma verify = <b>true</b>;
</code></pre>
