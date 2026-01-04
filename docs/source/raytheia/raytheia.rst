.. _sec-raytheia:

The RAYTHEIA algorithm
======================

RAYTHEIA is a high-performance reverse ray-tracing algorithm designed to efficiently solve three-dimensional direction-dependent equations in astronomical simulations. The algorithm is build on a uniform Cartesian grid combined with the HEALPix scheme for uniformly emanating rays across the celestial sphere.

Its key innovations include an octree-accelerated method for efficiently identifying grid-ray intersections, which reduces the time complexity of the search from :math:`{\cal O}(N^2)` to :math:`{\cal O}(N\log N)`, and an analytical slab method for the rapid, precise calculation of penetration lengths.

RAYTHEIA employs a dual-grid hybrid (MPI/OpenMP) distributed parallel framework with a chunk-to-chunk communication strategy, achieving exceptional, near-ideal linear speed-up ratio and delivering outsdanding performance.

You can download RAYTHEIA from this `Github link <https://github.com/itamos-ism/RAYTHEIA>`_. 
