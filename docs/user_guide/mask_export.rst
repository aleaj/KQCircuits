Mask export
===========

Masks are defined by simple scripts, which define the layout of chips in the
mask and the parameters of those chips. These scripts will export
``.oas``/``.gds`` files required for photomask production and possibly EBL,
as well as auxiliary files such as netlists, images and automatic mask
documentation. There are three ways to run the mask scripts:


- With the console script: ``kqc mask quick_demo.py``. This uses multiple processes for efficiency.
  This should be the default way of usage.
- As a standalone python script from the console in a single process for debugging: ``python
  ./scripts/masks/quick_demo.py -d`` or ``kqc mask quick_demo.py -d``. This also prints out debug
  information.
- From the KLayout macro editor. Slow single process execution, good to observe the created mask
  without loading it from file.

.. note::
    Windows and Mac needs the console script (``kqc``) to export a mask using multiprocessing but in Linux you may
    run them directly from the terminal with ``python scripts/masks/quick_demo.py``.

Tutorial
--------

Basic mask script and export
^^^^^^^^^^^^^^^^^^^^^^^^^^^^

    #. Launch KLayout Editor and open the Macro IDE (F5).

    #. Add the following script to the :git_url:`kqcircuits_scripts/masks <klayout_package/python/scripts/masks>` folder in
       the ``Python`` tab of the Macro IDE::

        # Imports required for any mask
        from kqcircuits.masks.mask_set import MaskSet
        # Import chips required for this mask
        from kqcircuits.chips.demo import Demo


        # Initialize mask object.
        # "name" and "version" will be displayed in labels inside the mask
        # and in the names of the exported files.
        # "with_grid" determines if the ground plane grid is created.
        test_mask = MaskSet(name="Test", version=1, with_grid=False)

        # Add mask layouts.
        # A mask layout determines which chip is at each position. The chips are
        # identified by strings which are defined by the `add_chip()` calls
        # below. "---" denotes places without a chip.
        test_mask.add_mask_layout([
            ["---", "---", "---", "---", "---", "---", "---", "---", "---", "---", "---", "---", "---", "---", "---"],
            ["---", "---", "---", "---", "---", "DE1", "DE1", "DE1", "DE1", "DE1", "---", "---", "---", "---", "---"],
            ["---", "---", "---", "DE1", "DE1", "DE1", "DE1", "DE1", "DE1", "DE1", "DE1", "DE1", "---", "---", "---"],
            ["---", "---", "DE1", "DE1", "DE1", "DE1", "DE1", "DE1", "DE1", "DE1", "DE1", "DE1", "DE1", "---", "---"],
            ["---", "---", "DE1", "DE1", "DE1", "DE1", "DE1", "DE1", "DE1", "DE1", "DE1", "DE1", "DE1", "---", "---"],
            ["---", "DE1", "DE1", "DE1", "DE1", "DE1", "DE1", "DE1", "DE1", "DE1", "DE1", "DE1", "DE1", "DE1", "---"],
            ["---", "DE1", "DE1", "DE1", "DE1", "DE1", "DE1", "DE1", "DE1", "DE1", "DE1", "DE1", "DE1", "DE1", "---"],
            ["---", "DE1", "DE1", "DE1", "DE1", "DE1", "DE1", "DE1", "DE1", "DE1", "DE1", "DE1", "DE1", "DE1", "---"],
            ["---", "DE1", "DE1", "DE1", "DE1", "DE1", "DE1", "DE1", "DE1", "DE1", "DE1", "DE1", "DE1", "DE1", "---"],
            ["---", "DE1", "DE1", "DE1", "DE1", "DE1", "DE1", "DE1", "DE1", "DE1", "DE1", "DE1", "DE1", "DE1", "---"],
            ["---", "---", "DE1", "DE1", "DE1", "DE1", "DE1", "DE1", "DE1", "DE1", "DE1", "DE1", "DE1", "---", "---"],
            ["---", "---", "DE1", "DE1", "DE1", "DE1", "DE1", "DE1", "DE1", "DE1", "DE1", "DE1", "DE1", "---", "---"],
            ["---", "---", "---", "DE1", "DE1", "DE1", "DE1", "DE1", "DE1", "DE1", "DE1", "DE1", "---", "---", "---"],
            ["---", "---", "---", "---", "---", "DE1", "DE1", "DE1", "DE1", "DE1", "---", "---", "---", "---", "---"],
            ["---", "---", "---", "---", "---", "---", "---", "---", "---", "---", "---", "---", "---", "---", "---"],
        ])

        # Chip definitions
        test_mask.add_chip(Demo, "DE1")

        # Build the mask.
        # This generates the layout in KLayout based on previous calls to
        # ``add_mask_layout()`` and ``add_chip()`.
        test_mask.build()
        # Export the mask files to TMP_PATH, unless an ``export_path`` argument has been given to MaskSet.
        test_mask.export()

       Any mask script should follow roughly the same structure. The
       beginning and end of the script (creating the MaskSet object, and exporting the
       mask) are generally the same for any mask.

    #. (Optional) Open :git_url:`kqcircuits_scripts/macros/logging_setup <klayout_package/python/scripts/macros/logging_setup.lym>` and run it.
       This makes mask generation logs visible in the Macro console.

    #. With the mask script open, click "Run script from current tab" (Shift+F5). This generates the
       mask and exports the files to the folder defined by ``kqcircuits.default.TMP_PATH`` or to any
       directory spcified by the ``export_path`` argument to ``MaskSet``.

If you generate the mask again (for example in the following sections), it is
recommended that you first close the layout where the previous mask was
generated.

Ground grid
^^^^^^^^^^^

The generated masks can include a ground plane grid, which automatically
avoids other elements. To generate it, use ``with_grid=True`` in the mask object
initialization::

    test_mask = MaskSet(name="Test", version=1, with_grid=True)

Because the grid generation is slow, it is recommended to keep
``with_grid=False`` when testing masks, and only use ``with_grid=True`` for
masks that are really produced.

Adding and modifying chips
^^^^^^^^^^^^^^^^^^^^^^^^^^

#. Add the following lines to the imports section of the script::

    from kqcircuits.chips.demo_twoface import DemoTwoface
    from kqcircuits.chips.quality_factor import QualityFactor

#. Add a new variant of the demo chip with different parameters::

    test_mask.add_chip(Demo, "DE2", include_couplers=False, readout_res_lengths=[5400, 5500, 5600, 5700])

#. Add several chips at once in parallel::

    test_mask.add_chip([
        (DemoTwoface, "DT1"),
        (QualityFactor, "QF1"),
    ])

#. You may also add chips generated elsewhere and stored in a .oas file::

    test_mask.add_chip("/home/user/my_chip.oas", "MCF")

#. Replace some of the "DE1" entries in the mask layout by "DT1", "QF1" and
   "DE2". This will determine where those chips are placed in the mask.

#. Generate the mask. You should see the new chip variants at the positions
   defined in the previous step.

Multi-face masks
^^^^^^^^^^^^^^^^

The mask generated in the previous step contained "DemoTwoface" chips, which
are supposed to have elements in two different chip faces for a flip-chip
process. However, the generated mask only contained the parts in "1t1"-face.
To generate mask for also "2b1"-face chips, we need to add an
additional :git_url:`mask_layout <klayout_package/python/kqcircuits/masks/mask_layout.py>`

#. Add the following below the old ``add_mask_layout()`` call::

    test_mask.add_mask_layout([
        ["---", "---", "---", "---", "---", "---", "---", "---", "---", "---", "---", "---", "---", "---", "---", "---", "---", "---", "---", "---", "---"],
        ["---", "---", "---", "---", "---", "---", "DT1", "DT1", "DT1", "DT1", "DT1", "DT1", "DT1", "DT1", "DT1", "---", "---", "---", "---", "---", "---"],
        ["---", "---", "---", "---", "---", "DT1", "DT1", "DT1", "DT1", "DT1", "DT1", "DT1", "DT1", "DT1", "DT1", "DT1", "---", "---", "---", "---", "---"],
        ["---", "---", "---", "---", "DT1", "DT1", "DT1", "DT1", "DT1", "DT1", "DT1", "DT1", "DT1", "DT1", "DT1", "DT1", "DT1", "---", "---", "---", "---"],
        ["---", "---", "---", "DT1", "DT1", "DT1", "DT1", "DT1", "DT1", "DT1", "DT1", "DT1", "DT1", "DT1", "DT1", "DT1", "DT1", "DT1", "---", "---", "---"],
        ["---", "---", "DT1", "DT1", "DT1", "DT1", "DT1", "DT1", "DT1", "DT1", "DT1", "DT1", "DT1", "DT1", "DT1", "DT1", "DT1", "DT1", "DT1", "---", "---"],
        ["---", "DT1", "DT1", "DT1", "DT1", "DT1", "DT1", "DT1", "DT1", "DT1", "DT1", "DT1", "DT1", "DT1", "DT1", "DT1", "DT1", "DT1", "DT1", "DT1", "---"],
        ["---", "DT1", "DT1", "DT1", "DT1", "DT1", "DT1", "DT1", "DT1", "DT1", "DT1", "DT1", "DT1", "DT1", "DT1", "DT1", "DT1", "DT1", "DT1", "DT1", "---"],
        ["---", "DT1", "DT1", "DT1", "DT1", "DT1", "DT1", "DT1", "DT1", "DT1", "DT1", "DT1", "DT1", "DT1", "DT1", "DT1", "DT1", "DT1", "DT1", "DT1", "---"],
        ["---", "DT1", "DT1", "DT1", "DT1", "DT1", "DT1", "DT1", "DT1", "DT1", "DT1", "DT1", "DT1", "DT1", "DT1", "DT1", "DT1", "DT1", "DT1", "DT1", "---"],
        ["---", "DT1", "DT1", "DT1", "DT1", "DT1", "DT1", "DT1", "DT1", "DT1", "DT1", "DT1", "DT1", "DT1", "DT1", "DT1", "DT1", "DT1", "DT1", "DT1", "---"],
        ["---", "DT1", "DT1", "DT1", "DT1", "DT1", "DT1", "DT1", "DT1", "DT1", "DT1", "DT1", "DT1", "DT1", "DT1", "DT1", "DT1", "DT1", "DT1", "DT1", "---"],
        ["---", "DT1", "DT1", "DT1", "DT1", "DT1", "DT1", "DT1", "DT1", "DT1", "DT1", "DT1", "DT1", "DT1", "DT1", "DT1", "DT1", "DT1", "DT1", "DT1", "---"],
        ["---", "DT1", "DT1", "DT1", "DT1", "DT1", "DT1", "DT1", "DT1", "DT1", "DT1", "DT1", "DT1", "DT1", "DT1", "DT1", "DT1", "DT1", "DT1", "DT1", "---"],
        ["---", "DT1", "DT1", "DT1", "DT1", "DT1", "DT1", "DT1", "DT1", "DT1", "DT1", "DT1", "DT1", "DT1", "DT1", "DT1", "DT1", "DT1", "DT1", "DT1", "---"],
        ["---", "---", "DT1", "DT1", "DT1", "DT1", "DT1", "DT1", "DT1", "DT1", "DT1", "DT1", "DT1", "DT1", "DT1", "DT1", "DT1", "DT1", "DT1", "---", "---"],
        ["---", "---", "---", "DT1", "DT1", "DT1", "DT1", "DT1", "DT1", "DT1", "DT1", "DT1", "DT1", "DT1", "DT1", "DT1", "DT1", "DT1", "---", "---", "---"],
        ["---", "---", "---", "---", "DT1", "DT1", "DT1", "DT1", "DT1", "DT1", "DT1", "DT1", "DT1", "DT1", "DT1", "DT1", "DT1", "---", "---", "---", "---"],
        ["---", "---", "---", "---", "---", "DT1", "DT1", "DT1", "DT1", "DT1", "DT1", "DT1", "DT1", "DT1", "DT1", "DT1", "---", "---", "---", "---", "---"],
        ["---", "---", "---", "---", "---", "---", "DT1", "DT1", "DT1", "DT1", "DT1", "DT1", "DT1", "DT1", "DT1", "---", "---", "---", "---", "---", "---"],
        ["---", "---", "---", "---", "---", "---", "---", "---", "---", "---", "---", "---", "---", "---", "---", "---", "---", "---", "---", "---", "---"],
    ], "2b1")

   Notice the "2b1" argument after the actual mask layout. This means that only
   elements which are in chip face "2b1" will be included in this mask layout.
   By default, the mask layout will use elements in chip face "1t1", which is
   the case for the first mask layout in this script.

#. Generate the mask. In KLayout cell hierarchy, you should see both "Test 1t1"
   and "Test 2b1" cells. You can right click them and select "Show as new
   top" to verify that "Test 1t1" contains the parts of DemoTwoface in
   "1t1"-face and "Test 2b1" contains the parts of DemoTwoface in "2b1"-face
   (with proper mirroring and rotation).
