.. _apple_codesign_pkcs11:

====================
PKCS#11 (HSM) Support
====================

This project supports integration with PKCS#11-compatible Hardware
Security Modules (HSMs) and software tokens. This enables cryptographic
signing using certificates and private keys stored in secure hardware,
such as Amazon CloudHSM or Google Cloud HSM.

PKCS#11 integration is useful for organizations that require
high-assurance key management and want to keep private keys out of
software memory.

Cargo Feature
=============

PKCS#11 integration requires the optional and disabled-by-default
`pkcs11` Cargo feature to be enabled:

.. code-block:: shell

    cargo build --features pkcs11

Supported Devices
=================

Any device or software that provides a PKCS#11 interface should work. This includes:

- Amazon CloudHSM
- Google Cloud HSM
- SoftHSM (for testing)

You will need the vendor's PKCS#11 library (e.g.,
``/opt/cloudhsm/lib/libcloudhsm_pkcs11.so`` for Amazon, or the path
provided by Google for Cloud HSM).

Validating PKCS#11 Integration
==============================

To see if your HSM is recognized and certificates can be found, use your
certificate as a file:

.. code-block:: shell

    $ rcodesign sign --pkcs11-library /path/to/pkcs11.so --pkcs11-pin <PIN> --pkcs11-certificate-file /path/to/cert.pem --pkcs11-key-label <LABEL> <file-to-sign>

You can also use ``--pkcs11-slot-id`` or ``--pkcs11-token-label`` to select a specific slot or token.

Amazon CloudHSM Example
=======================

1. Ensure your CloudHSM cluster is initialized and you have a user and key/certificate loaded.
2. Set up the CloudHSM client and note the path to the PKCS#11 library (usually ``/opt/cloudhsm/lib/libcloudhsm_pkcs11.so``).
3. Sign a file (using the certificate file issued by Apple or your CA):

.. code-block:: shell

    rcodesign sign \
        --pkcs11-library /opt/cloudhsm/lib/libcloudhsm_pkcs11.so \
        --pkcs11-pin <USER:PASS> \
        --pkcs11-certificate-file /path/to/cert.pem \
        --pkcs11-key-label <KEY_LABEL> \
        <file-to-sign>

Google Cloud HSM Example
========================

1. Set up your Google Cloud HSM and install the PKCS#11 library (see Google documentation for the path).
2. Ensure your key and certificate are loaded and note their labels or IDs.
3. Sign a file (using the certificate file issued by Apple or your CA):

.. code-block:: shell

    rcodesign sign \
        --pkcs11-library /path/to/google/pkcs11.so \
        --pkcs11-pin <USER_PIN> \
        --pkcs11-certificate-file /path/to/cert.pem \
        --pkcs11-key-label <KEY_LABEL> \
        <file-to-sign>

Testing with SoftHSM
====================

For development and testing, you can use SoftHSM:

.. code-block:: shell

    # Initialize a new token
    softhsm2-util --init-token --slot 0 --label "TestToken"

    # Import a key and certificate (see SoftHSM docs)

    # Use rcodesign with SoftHSM's PKCS#11 library
    rcodesign sign \
        --pkcs11-library /usr/local/lib/softhsm/libsofthsm2.so \
        --pkcs11-pin <PIN> \
        --pkcs11-certificate-file /path/to/cert.pem \
        --pkcs11-key-label <KEY_LABEL> \
        <file-to-sign>

Limitations
===========

- You must know the correct label or ID for your key in the HSM, and
  have the certificate file available (unless you have imported the
  certificate into the HSM, which is uncommon).
- Some HSMs require additional configuration or environment variables.
- Only signing is supported; key generation and import must be done
  using vendor tools.
