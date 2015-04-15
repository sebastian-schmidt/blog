.. title: Adobe password leak analyzed with pandas and Python
.. slug: adobe-password-leak-analyzed-with-pandas-and-python
.. date: 2015-04-15 22:38:53 UTC+02:00
.. tags:
.. category:
.. link:
.. description:
.. type: text

I recently found out that one of my throwaway email-accounts was 
in the Adobe password leak. I wanted to see what was 
out there and downloaded the leaked data. The leaked data looks like this:

::

	<some ID>-|-<username, mostly missing>-|-<email>-|-<password hash>-|-<password hint>|--
	103238705-|--|-asdgagr@yahoo.com-|-BB4e6X+b2xLioxG6CatHBw==-|-boyfriend|--
	103238706-|--|-fasggfso@hotmail.com-|-Cm8mAzxAiwzioxG6CatHBw==-|-dance|--
	103238707-|--|-witdfhadki@gmail.com-|-n+TZlu41zyHioxG6CatHBw==-|-|--
	103238708-|--|-isdfahdh8@gmail.com-|-FAniAwP+U13ioxG6CatHBw==-|-|--
	103238709-|--|-ojaadf@yahoo.com-|-kxiV+a47bSlf+E5Ulu/AzA==-|-newest|--
	103238710-|--|-sahjg@hotmail.com-|-UimSy9NunUU=-|-dog|--

I hacked

.. 