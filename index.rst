..
  Technote content.

  See https://developer.lsst.io/restructuredtext/style.html
  for a guide to reStructuredText writing.

  Do not put the title, authors or other metadata in this document;
  those are automatically added.

  Use the following syntax for sections:

  Sections
  ========

  and

  Subsections
  -----------

  and

  Subsubsections
  ^^^^^^^^^^^^^^

  To add images, add the image file (png, svg or jpeg preferred) to the
  _static/ directory. The reST syntax for adding the image is

  .. figure:: /_static/filename.ext
     :name: fig-label

     Caption text.

   Run: ``make html`` and ``open _build/html/index.html`` to preview your work.
   See the README at https://github.com/lsst-sqre/lsst-technote-bootstrap or
   this repo's README for more info.

   Feel free to delete this instructional comment.

:tocdepth: 1

.. Please do not modify tocdepth; will be fixed when a new Sphinx theme is shipped.

.. sectnum::

.. TODO: Delete the note below before merging new content to the master branch.

.. note::

   **This technote is not yet published.**

   This document provides suggestions to consider when contemplating design enhancements for the nublado system.

.. Add content here.
.. Do not include the document title (it's automatically added from metadata.yaml).

Introduction
============

This document should be read as a list of suggestions or recommendations when contemplating design or architecture decisions related to the nublado system.
Even though some of these statements use requirements language, they are not requirements and may be reconsidered in the face of developing technologies and the wider ecosystem as a whole.
The expectation is that most of these will be addressed via already known technologies, and in fact many are already met by the running system.
The intent was more to capture functionality that is not specifically required by the formal requirements, but that is all the same recognized as being important to the functionality of the system as a whole.

Recommendations
===============

Following are a series of recommendations that have been grouped into several categories: **UX**, **Performance**, **A&A**, and **Architecture**.
Some may touch on several aspects.
In those cases, they typically were placed in the **Architecture** despite also have impacts on e.g. **UX**.

UX:
^^^

* Limitations of the terminal should be understood and clearly communicated.
  This includes things like keystrokes being intercepted by the browser.
* Pop-up windows should be used sparingly and only with information critical to the users understanding of the current environment or action.
  Additionally, information in pop-ups should always be consistent with current system status: e.g. the user should not be informed the server is not running when, in fact, the status of the server can't be determined.
* Desktop apps do not need be supported: e.g. topcat, DS9, or word.
* A fixed set of editing environments will be supported: ed, nano, vim, JupyterLab plugin text editor, EMACS (possibly with caveats).
  This should not be seen as normative, but the final set of supported editors should be clearly documented.
* There should be a user visible, independently hosted health dashboard like https://www.githubstatus.com.
* There should be an announcement mechanism to advertise changes and outages.
* State should be preserved to the extent possible between log in actions.
  Notebooks should open connected to a running kernel.
  Terminals should open at a fresh prompt.
  To the extent possible, it should be clear that no cells in the notebook were run on startup.
* An image display tool should be available as a JupyterLab plugin so that things like double clicking FITS files in the file browser open automatically.

Performance:
^^^^^^^^^^^^

* There should be at least one performant plotting utility available within the notebooks.
  This may evolve, but rendering a 2-D scatter plot with :math:`10^5` points should render in under a second.
* There should be a way to plot more than :math:`10^6` points within a single container in a performant way.
  This could use data aggregation technologies like ``datashader``.
  There could be a staging step, but once rendered, interaction should be near seamless.
  Pan and zoom operations should take less than 0.5 seconds to update.
* The included interactive image environment should be equally performant.
  Loading images may take a few seconds, but once loaded pan and zoom operations should be similarly performant to visualizing aggregated scatter plots.
  Adjusting scaling should be natural, preferentially using gestures.
  Pixel, and potentially mask, values should update continuously taking no more than 0.05 sec to update from pixel to pixel.
* Nublado should provide packages that natively allow for basic "drill down" activities including brushing and linking and interactive metadata discovery (via e.g. hover).
  It should be possible to discover data identifiers for images associated with catalog data points.

A&A:
^^^^

* If resources become over topped, the login process must be graceful and informative in the face of additional login attempts.
  We need to be able to turn away users in a pleasant way.
* The login mechanism should support 5 successful logon attempts per second.
  Rates higher than can be handled by the login machinery should be queued until load decreases.

Administration:
^^^^^^^^^^^^^^^

* It should be possible for users to be able to provide information when reporting a problem to a help desk: e.g. running node, stack version number, installed packages.
* It should be possible for administrators to get the same information as above without user intervention: e.g. running node, stack version number, installed packages.
* There should be an administration health dashboard requirement.
  This includes both cluster information as well as k8s information.
  It should be easy to discover the status of the pods belonging to a particular user.
* There should be a mechanism to reset all environment files (.bash, .local, etc) to default values on spawn.
  This is to allow users to recover if their environment is in a bad state.
  This should preserve, as much as possible, the existing user environment not installed by the service at first spawn time.

Architecture:
^^^^^^^^^^^^^

* The architecture of nublado should use JupyterLab at its core.
  This means that the architecture should be structured such that upgrading JupyterLab should be as easy as possible.
* Any user action that degrades service performance should be able to be mitigated by the operations team without any kind of intervention from the user.
  This means we should not need to ask users to close tabs in order to get readable logs back.
* Things like mobu or workflow should be possible: i.e. simulating user interactions and notebook CI driven through a programmatic mechanism.
* The system should gracefully and automatically scale to the number of users the compute resources permit up to 8000 users.
* Even if gated, the deployment chain should be GitOps based.
  This includes production of any images or packages utilized by the deployment process.
* On the basis of group membership access to the systems resources should be able to be controlled differently.
* Pod lifetime should be enforceable (this means it needs to be possible to implement a culler).
* When spawning as an authorized user on a node with the relevant image existing in node local cache, it shall take no more than 20 seconds to reach a running JupyterLab from the landing page on average.
  This implies we need something like a prepuller to make sure large files are cached on the nodes where users spawn.
* The prepuller should be implemented in a way such that I/O load is spread as evenly as possible.
  Ideally this means that nodes would try not to prepull at the same time.
* User authentication and pod lifetimes must be handled such that either users never see authentication expiry with a running pod or that re-authentication in that scenario leads to the user re-entering the container gracefully.
* Users in the nublado environmet should not expect access to databases via bare SQL queries.
* The architecture should prohibit authenticated users from subverting user or group based permissions.
  Authenticated users should not be able to see data they do not own or have not been given explicit access to.
  They should not be able to see inside other user's containers.
  They should not be able to utilize more than their authorized compute budget.
  This list is not exhaustive.
  Note this specifically speaks to authenticated users.
  This system is not responsible for defending against arbitrary breaches of the underlying A&A system.

.. .. rubric:: References

.. Make in-text citations with: :cite:`bibkey`.

.. .. bibliography:: local.bib lsstbib/books.bib lsstbib/lsst.bib lsstbib/lsst-dm.bib lsstbib/refs.bib lsstbib/refs_ads.bib
..    :style: lsst_aa
