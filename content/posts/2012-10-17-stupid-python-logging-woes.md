+++
date = "2012-10-17T20:11:00+00:00"
draft = false
tags = ["coding"]
title = "stupid Python logging woes"
+++
Recently moved a project to Django 1.3 and I kept seeing duplicate messages for all of my logger.x messages, even though my LOGGING dictionary looked like:


    LOGGING = {
        'version': 1,
        'disable_existing_loggers': True,
        'loggers': {
            'base': {
                'level': 'DEBUG',
            },
        }
    }

I simplified it to this to see if there were any issues, and although I define no handlers for the 'base' logger, it still was printing out logging messages. I did a grep on the entire virtualenv libraries for basicConfig or BASIC_FORMAT to see if any of my external libraries were misbehaving and didn't find anything. I went through my codebase again and got rid of some extraneous logging.debug in some files and that fixed it. So always use 

    import logging
    logger = logging.getLogger(__name__)
    logger.debug(....)

instead of logging.debug directly!