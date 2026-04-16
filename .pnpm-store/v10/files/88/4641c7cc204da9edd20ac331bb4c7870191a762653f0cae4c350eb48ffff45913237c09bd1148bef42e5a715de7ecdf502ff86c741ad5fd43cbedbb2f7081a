"use strict";

Object.defineProperty(exports, "__esModule", {
    value: true
});

var _typeof = typeof Symbol === "function" && typeof Symbol.iterator === "symbol" ? function (obj) { return typeof obj; } : function (obj) { return obj && typeof Symbol === "function" && obj.constructor === Symbol && obj !== Symbol.prototype ? "symbol" : typeof obj; };

exports.default = polyfill;

var _lookupUtill = require("./lookup-utill.js");

function _get(object, property, receiver) {
    if (object === null) object = Function.prototype;
    var desc = Object.getOwnPropertyDescriptor(object, property);
    if (desc === undefined) {
        var parent = Object.getPrototypeOf(object);
        if (parent === null) {
            return undefined;
        }
        return _get(parent, property, receiver);
    }
    if ("value" in desc) {
        return desc.value;
    }
    var getter = desc.get;
    if (getter === undefined) {
        return undefined;
    }
    return getter.call(receiver);
}

function _getPrototypeOf(Class) {
    return Object.getPrototypeOf ? Object.getPrototypeOf(Class) : Class.__proto__;
}

function _setPrototypeOf(subClass, superClass) {
    if (Object.setPrototypeOf) {
        return Object.setPrototypeOf(subClass, superClass);
    }
    subClass.__proto__ = superClass;
    return subClass;
}

function _isNativeFunction(fn) {
    return Function.toString.call(fn).indexOf("[native code]") !== -1;
}

function _wrapNativeSuper(Class) {
    if (!_isNativeFunction(Class)) {
        return Class;
    }
    function Wrapper() {
        var prototypeConstructor = _getPrototypeOf(this).constructor;
        var args = [null];
        args.push.apply(args, arguments);
        var Constructor = Function.bind.apply(Class, args);
        var instance = new Constructor();
        if (prototypeConstructor) {
            _setPrototypeOf(instance, prototypeConstructor.prototype);
        }
        return instance;
    }
    Wrapper.prototype = Object.create(Class.prototype, {
        constructor: {
            value: Wrapper,
            enumerable: false,
            writable: true,
            configurable: true
        }
    });
    return _setPrototypeOf(Wrapper, Class);
}

function _inherits(subClass, superClass) {
    if (typeof superClass !== "function" && superClass !== null) {
        throw new TypeError("Super expression must either be null or a function, not " + (typeof superClass === "undefined" ? "undefined" : _typeof(superClass)));
    }
    subClass.prototype = Object.create(superClass && superClass.prototype, {
        constructor: {
            value: subClass,
            enumerable: false,
            writable: true,
            configurable: true
        }
    });
    if (superClass) {
        _setPrototypeOf(subClass, superClass);
    }
}

function polyfill(globalSpace) {
    if (!(globalSpace.Intl && globalSpace.Intl.DateTimeFormat && !globalSpace.Intl._DateTimeFormatTimeZone)) {
        return;
    }

    var gIntl = globalSpace.Intl;
    var gDate = globalSpace.Date;
    var checkTimeZoneSupport = (0, _lookupUtill.buildCachedCheckTimeZoneSupport)(globalSpace);
    var jsonClone = function jsonClone(o) {
        return JSON.parse(JSON.stringify(o));
    };
    var _DateTimeFormat = _wrapNativeSuper(gIntl.DateTimeFormat);

    gIntl._DateTimeFormat = _DateTimeFormat;

    gIntl._DateTimeFormatTimeZone = {
        checkTimeZoneSupport: checkTimeZoneSupport
    };

    function DateTimeFormatPolyfill(locale, options) {
        if (!(this instanceof DateTimeFormatPolyfill)) {
            return new DateTimeFormatPolyfill(locale, options);
        }

        var _this = void 0;

        var timeZone = options && options.timeZone || 'UTC';

        if (options === undefined) {
            _this = _DateTimeFormat.call(this, locale, options);

            if (_this.formatToParts) {
                _this._nativeObject = new _DateTimeFormat(locale, options);
            }

            return _this;
        }

        if (checkTimeZoneSupport(timeZone)) {
            _this = _DateTimeFormat.call(this, locale, options);

            if (_this.formatToParts) {
                _this._nativeObject = new _DateTimeFormat(locale, options);
            }

            return _this;
        }

        var timeZoneData = gIntl._timeZoneData.get(timeZone);

        if (!timeZoneData) {
            throw new RangeError("invalid time zone in DateTimeFormat():  " + timeZone);
        }

        var tsOption = jsonClone(options);
        tsOption.timeZone = 'UTC';
        _this = _DateTimeFormat.call(this, locale, tsOption);

        var _resolvedOptions = _get(_getPrototypeOf(DateTimeFormatPolyfill.prototype), 'resolvedOptions', _this);

        var resolvedLocale = _resolvedOptions.call(_this).locale;

        if (options.timeZoneName !== undefined) {
            if (!(gIntl._localeData.getLocale(resolvedLocale) && Intl._metaZoneData.get(timeZone))) {
                throw new RangeError("unsupported value \"" + options.timeZoneName + "\" for timeZone " + timeZone + ". requires locale data for " + resolvedLocale);
            }
        }

        _this._dateTimeFormatPolyfill = {
            optionTimeZone: timeZone,
            optionTimeZoneName: options.timeZoneName,
            timeZoneData: timeZoneData
        };

        return _this;
    }
    _inherits(DateTimeFormatPolyfill, _DateTimeFormat);

    Object.defineProperty(DateTimeFormatPolyfill.prototype, 'format', {
        configurable: true,
        value: function value(date) {
            var _format = _get(_getPrototypeOf(DateTimeFormatPolyfill.prototype), 'format', this);
            var _resolvedOptions = _get(_getPrototypeOf(DateTimeFormatPolyfill.prototype), 'resolvedOptions', this);

            if (!this._dateTimeFormatPolyfill) {
                return _format.call(this, date);
            }

            if (date === null || date === undefined) {
                date = new Date();
            }

            if (!(date instanceof Date)) {
                date = new Date(date);
            }

            var polyfill = this._dateTimeFormatPolyfill;
            var timeZoneOffsetInfo = (0, _lookupUtill.getTimeZoneOffsetInfo)(polyfill.timeZoneData, date);
            var timeZoneOffset = timeZoneOffsetInfo.offset * 60000;
            var shiftedDate = new Date(date.getTime() + timeZoneOffset);
            var shiftedFormat = _format.call(this, shiftedDate);
            var resolvedLocale = _resolvedOptions.call(this).locale;
            var doNeedToReplaceTimeZoneName = polyfill.optionTimeZoneName !== undefined;

            if (doNeedToReplaceTimeZoneName) {
                var isShort = polyfill.optionTimeZoneName === 'short';
                var timeZoneName = (0, _lookupUtill.getZoneNameForLocale)({
                    locale: resolvedLocale,
                    ianaTimeZone: polyfill.optionTimeZone,
                    isdst: timeZoneOffsetInfo.isdst,
                    offset: timeZoneOffsetInfo.offset,
                    timeStamp: date.getTime(),
                    isShort: isShort
                });
                var timeZoneNameUTC = (0, _lookupUtill.getZoneNameForLocale)({
                    locale: resolvedLocale,
                    ianaTimeZone: 'UTC',
                    isdst: false,
                    offset: 0,
                    timeStamp: date.getTime(),
                    isShort: isShort
                });

                if (shiftedFormat.indexOf(timeZoneNameUTC) < 0) {
                    return shiftedFormat.trim() + ' ' + timeZoneName;
                }

                return shiftedFormat.replace(timeZoneNameUTC, timeZoneName);
            }

            return shiftedFormat;
        }
    });

    var _formatToParts = _get(_getPrototypeOf(DateTimeFormatPolyfill.prototype), 'formatToParts', this);

    if (_formatToParts) {
        Object.defineProperty(DateTimeFormatPolyfill.prototype, 'formatToParts', {
            configurable: true,
            value: function value(date) {
                var _resolvedOptions = _get(_getPrototypeOf(DateTimeFormatPolyfill.prototype), 'resolvedOptions', this);

                if (!this._dateTimeFormatPolyfill && this._nativeObject) {
                    return this._nativeObject.formatToParts(date);
                }

                if (date === null || date === undefined) {
                    date = new Date();
                }

                if (!(date instanceof Date)) {
                    date = new Date(date);
                }

                var polyfill = this._dateTimeFormatPolyfill;
                var timeZoneOffsetInfo = (0, _lookupUtill.getTimeZoneOffsetInfo)(polyfill.timeZoneData, date);
                var timeZoneOffset = timeZoneOffsetInfo.offset * 60000;
                var shiftedDate = new Date(date.getTime() + timeZoneOffset);
                var shiftedParts = _formatToParts.call(this, shiftedDate);
                var resolvedLocale = _resolvedOptions.call(this).locale;
                var doNeedToReplaceTimeZoneName = polyfill.optionTimeZoneName !== undefined;

                if (doNeedToReplaceTimeZoneName) {
                    var isShort = polyfill.optionTimeZoneName === 'short';
                    var timeZoneName = (0, _lookupUtill.getZoneNameForLocale)({
                        locale: resolvedLocale,
                        ianaTimeZone: polyfill.optionTimeZone,
                        isdst: timeZoneOffsetInfo.isdst,
                        offset: timeZoneOffsetInfo.offset,
                        timeStamp: date.getTime(),
                        isShort: isShort
                    });

                    var index = shiftedParts.map(function (i) {
                        return i.type;
                    }).indexOf('timeZoneName');

                    if (index >= 0) {
                        shiftedParts[index] = {
                            type: 'timeZoneName',
                            value: timeZoneName
                        };
                    }
                }

                return shiftedParts;
            }
        });
    }

    Object.defineProperty(DateTimeFormatPolyfill.prototype, 'resolvedOptions', {
        writable: true,
        configurable: true,
        value: function value() {
            var _resolvedOptions = _get(_getPrototypeOf(DateTimeFormatPolyfill.prototype), 'resolvedOptions', this);

            if (this._dateTimeFormatPolyfill) {
                var options = jsonClone(_resolvedOptions.call(this));
                options.timeZone = this._dateTimeFormatPolyfill.optionTimeZone;

                return options;
            }

            return _resolvedOptions.call(this);
        }
    });

    gIntl.DateTimeFormat = DateTimeFormatPolyfill;

    gDate.prototype.toLocaleString = function (locale, options) {
        var defaultLocaleOption = {
            day: 'numeric',
            month: 'numeric',
            year: 'numeric',
            hour: 'numeric',
            minute: 'numeric',
            second: 'numeric'
        };

        if (options === undefined) {
            options = jsonClone(defaultLocaleOption);
        }

        if (options.day === undefined && options.month === undefined && options.year === undefined && options.hour === undefined && options.minute === undefined && options.second === undefined) {
            options = jsonClone(options);
            options.day = defaultLocaleOption.day;
            options.month = defaultLocaleOption.month;
            options.year = defaultLocaleOption.year;
            options.hour = defaultLocaleOption.hour;
            options.minute = defaultLocaleOption.minute;
            options.second = defaultLocaleOption.second;
        }

        return new gIntl.DateTimeFormat(locale, options).format(this);
    };

    gDate.prototype.toLocaleDateString = function (locale, options) {
        var defaultDateOption = {
            day: 'numeric',
            month: 'numeric',
            year: 'numeric'
        };

        if (options === undefined) {
            options = jsonClone(defaultDateOption);
        }

        if (options.day === undefined && options.month === undefined && options.year === undefined) {
            options = jsonClone(options);
            options.day = defaultDateOption.day;
            options.month = defaultDateOption.month;
            options.year = defaultDateOption.year;
        }

        return new gIntl.DateTimeFormat(locale, options).format(this);
    };

    gDate.prototype.toLocaleTimeString = function (locale, options) {
        var defaultTimeOption = {
            hour: 'numeric',
            minute: 'numeric',
            second: 'numeric'
        };

        if (options === undefined) {
            options = jsonClone(defaultTimeOption);
        }

        if (options.hour === undefined && options.minute === undefined && options.second === undefined) {
            options = jsonClone(options);
            options.hour = defaultTimeOption.hour;
            options.minute = defaultTimeOption.minute;
            options.second = defaultTimeOption.second;
        }

        return new gIntl.DateTimeFormat(locale, options).format(this);
    };
}
//# sourceMappingURL=polyfill.js.map
