..
 This work is licensed under a Creative Commons Attribution 3.0 Unported
 License.

 http://creativecommons.org/licenses/by/3.0/legalcode

==========================================
XenAPI: Add support for vGPU
==========================================

https://blueprints.launchpad.net/nova/+spec/xenapi-add-support-for-vgpu

Add the function in xenapi to support vGPU in VMs which run on the hosts
which have remaining capacity of vGPU.

Problem description
===================

Current nova implement is NOT supporting vGPU; but the end-user may expect to
create VMs which are equiped with vGPU to achieve better graphics processing
capability.

Use Cases
----------

The end-user may create a VM  instance equiped with a vGPU. The user can
specify the vGPU's model type in flavor with the property of extra_specs.
Nova schedules hosts basing on the remaininig vGPU capacity and boot a VM
with a vGPU equipped.

Project Priority
-----------------

None

Proposed change
===============

1. Add function to query the vGPU capacity information from the host;
2. Add a new field of 'vgpus' in to the table of compute_nodes to hold
   the vGPU capacity information.
3. No change is needed to the scheduler; but the existing function of
   compute_capabilities_filter will be used to filter hosts basing on
   the key(capabilities:vgpus:<vGPU model>) specified in the flavor
   extra_specs.
4. Add new funtions to xenapi's instance spawn function to check the
   flavor's extra_specs. If a vGPU is required, it creates a vGPU with
   the requested model and attaches it to the new VM instance.

Alternatives
------------

None

Data model impact
-----------------

Need add a new text field to the class of ComputeNode and HostState to hold
the vgpu capacity information;
Need add a new Text typed column in the DB table of compute_nodes to save the
vgpu capacity information in nova database.

REST API impact
---------------

The change is transparent to REST API.

Security impact
---------------

None

Notifications impact
--------------------

None

Other end user impact
---------------------

If the end user wants to create a VM which has the GPU capability; the user
should add the key=value pair into the flavor's extra_specs properity:

.. code::
  e.g. set the key=value pair as 'capabilities:vgpus:<vGPU model-name>'='> 0'

which means the VM to be created by using this flavor should have a vGPU with
the model-name as <vGPU model-name>.

Performance Impact
------------------

<?
I'm not sure if we should mention that it may cost a little more time due to
the new XenAPI calling to retrieve vGPU info from host
?>

Other deployer impact
---------------------

None

Developer impact
----------------

None


Implementation
==============

See `Proposed change`_ section above.

Assignee(s)
-----------

Primary assignee:
  bobba

Other contributors:
  jianghuaw

Work Items
----------

Changes would be made, in order to:

1. retrieve the remaining vGPU capacity from the host;
2. create a new field into the DB table of compute_nodes;
3. check the extra_specs to determine whether and which type of vGPU should
   be created for the VM instance;
4. check the host's vGPU capacity and create requested vGPU when spawn a VM.

Dependencies
============

<?
depend on XenServer release which has the vGPU XenAPI supported. Or,
we handle it in code (which I mean to skip retrieving vGPU if we can
determine the XenServer is too old to support vGPU)
?>


Testing
=======

Would need new unit tests to cover host filter basing vGPU capacity and VM
generation with vGPU.


Documentation Impact
====================

May need document how to specify the vGPU model in the flavor extra_specs.
It should be similar as:

.. code::

  'capabilities:vgpus:<vGPU model-name>'='> 0'


References
==========

The prototype code:
https://review.openstack.org/#/c/223426/


History
=======

