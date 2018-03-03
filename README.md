# Mailman PostAuth

This module provides the additional confirmation step for the mailng
list servers using Mailman.  This module mainly consists of two
programs.

 * The controller CGI script: action
 * The mail handler script: mmpostauth-handler

The above two programs implements the following procedure:

 1. When a non-member sender posts a mail to the mailing list server
	using Mailman, the mail handler script transfers a moderation
	message of Mailman to the controller CGI script.
 2. The controller CGI script generates a temporally confirmation URL
    and sends a mail which notifies it to the original non-member
    sender.
 3. If someone accesses the confirmation URL, the orignal non-member
    sender is considered as a real human.  After that, the controller
    CGI script sends an approval command mail to the mailing list
    server, and the mailing list server delivers the mail posted by
    the orignal non-member sender.
