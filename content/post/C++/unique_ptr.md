---
title: "unique_ptr"
date: 2020-08-29T17:43:10+08:00
draft: true
---

# `unique_ptr`

```c++
template <typename _Tp, typename _Dp = default_delete<_Tp>>
    class unique_ptr
    {
      template <class _Up>
      using _DeleterConstraint =
	typename __uniq_ptr_impl<_Tp, _Up>::_DeleterConstraint::type;

      __uniq_ptr_impl<_Tp, _Dp> _M_t;

    public:
      using pointer	  = typename __uniq_ptr_impl<_Tp, _Dp>::pointer;
      using element_type  = _Tp;
      using deleter_type  = _Dp;

      // helper template for detecting a safe conversion from another
      // unique_ptr
      template<typename _Up, typename _Ep>
	using __safe_conversion_up = __and_<
	  is_convertible<typename unique_ptr<_Up, _Ep>::pointer, pointer>,
	  __not_<is_array<_Up>>
        >;

      // Constructors.

      /// Default constructor, creates a unique_ptr that owns nothing.
      template <typename _Up = _Dp,
		typename = _DeleterConstraint<_Up>>
	constexpr unique_ptr() noexcept
	: _M_t()
	{ }

      /** Takes ownership of a pointer.
       *
       * @param __p  A pointer to an object of @c element_type
       *
       * The deleter will be value-initialized.
       */
      template <typename _Up = _Dp,
		typename = _DeleterConstraint<_Up>>
	explicit
	unique_ptr(pointer __p) noexcept
	: _M_t(__p)
        { }

      /** Takes ownership of a pointer.
       *
       * @param __p  A pointer to an object of @c element_type
       * @param __d  A reference to a deleter.
       *
       * The deleter will be initialized with @p __d
       */
      unique_ptr(pointer __p,
	  typename conditional<is_reference<deleter_type>::value,
	    deleter_type, const deleter_type&>::type __d) noexcept
      : _M_t(__p, __d) { }

      /** Takes ownership of a pointer.
       *
       * @param __p  A pointer to an object of @c element_type
       * @param __d  An rvalue reference to a deleter.
       *
       * The deleter will be initialized with @p std::move(__d)
       */
      unique_ptr(pointer __p,
	  typename remove_reference<deleter_type>::type&& __d) noexcept
      : _M_t(std::move(__p), std::move(__d))
      { static_assert(!std::is_reference<deleter_type>::value,
		      "rvalue deleter bound to reference"); }

      /// Creates a unique_ptr that owns nothing.
      template <typename _Up = _Dp,
		typename = _DeleterConstraint<_Up>>
	constexpr unique_ptr(nullptr_t) noexcept : _M_t() { }

      // Move constructors.

      /// Move constructor.
      unique_ptr(unique_ptr&& __u) noexcept
      : _M_t(__u.release(), std::forward<deleter_type>(__u.get_deleter())) { }

      /** @brief Converting constructor from another type
       *
       * Requires that the pointer owned by @p __u is convertible to the
       * type of pointer owned by this object, @p __u does not own an array,
       * and @p __u has a compatible deleter type.
       */
      template<typename _Up, typename _Ep, typename = _Require<
               __safe_conversion_up<_Up, _Ep>,
	       typename conditional<is_reference<_Dp>::value,
				    is_same<_Ep, _Dp>,
				    is_convertible<_Ep, _Dp>>::type>>
	unique_ptr(unique_ptr<_Up, _Ep>&& __u) noexcept
	: _M_t(__u.release(), std::forward<_Ep>(__u.get_deleter()))
	{ }

#if _GLIBCXX_USE_DEPRECATED
      /// Converting constructor from @c auto_ptr
      template<typename _Up, typename = _Require<
	       is_convertible<_Up*, _Tp*>, is_same<_Dp, default_delete<_Tp>>>>
	unique_ptr(auto_ptr<_Up>&& __u) noexcept;
#endif

      /// Destructor, invokes the deleter if the stored pointer is not null.
      ~unique_ptr() noexcept
      {
	auto& __ptr = _M_t._M_ptr();
	if (__ptr != nullptr)
	  get_deleter()(__ptr);
	__ptr = pointer();
      }

      // Assignment.

      /** @brief Move assignment operator.
       *
       * @param __u  The object to transfer ownership from.
       *
       * Invokes the deleter first if this object owns a pointer.
       */
      unique_ptr&
      operator=(unique_ptr&& __u) noexcept
      {
	reset(__u.release());
	get_deleter() = std::forward<deleter_type>(__u.get_deleter());
	return *this;
      }

      /** @brief Assignment from another type.
       *
       * @param __u  The object to transfer ownership from, which owns a
       *             convertible pointer to a non-array object.
       *
       * Invokes the deleter first if this object owns a pointer.
       */
      template<typename _Up, typename _Ep>
        typename enable_if< __and_<
          __safe_conversion_up<_Up, _Ep>,
          is_assignable<deleter_type&, _Ep&&>
          >::value,
          unique_ptr&>::type
	operator=(unique_ptr<_Up, _Ep>&& __u) noexcept
	{
	  reset(__u.release());
	  get_deleter() = std::forward<_Ep>(__u.get_deleter());
	  return *this;
	}

      /// Reset the %unique_ptr to empty, invoking the deleter if necessary.
      unique_ptr&
      operator=(nullptr_t) noexcept
      {
	reset();
	return *this;
      }

      // Observers.

      /// Dereference the stored pointer.
      typename add_lvalue_reference<element_type>::type
      operator*() const
      {
	__glibcxx_assert(get() != pointer());
	return *get();
      }

      /// Return the stored pointer.
      pointer
      operator->() const noexcept
      {
	_GLIBCXX_DEBUG_PEDASSERT(get() != pointer());
	return get();
      }

      /// Return the stored pointer.
      pointer
      get() const noexcept
      { return _M_t._M_ptr(); }

      /// Return a reference to the stored deleter.
      deleter_type&
      get_deleter() noexcept
      { return _M_t._M_deleter(); }

      /// Return a reference to the stored deleter.
      const deleter_type&
      get_deleter() const noexcept
      { return _M_t._M_deleter(); }

      /// Return @c true if the stored pointer is not null.
      explicit operator bool() const noexcept
      { return get() == pointer() ? false : true; }

      // Modifiers.

      /// Release ownership of any stored pointer.
      pointer
      release() noexcept
      {
	pointer __p = get();
	_M_t._M_ptr() = pointer();
	return __p;
      }

      /** @brief Replace the stored pointer.
       *
       * @param __p  The new pointer to store.
       *
       * The deleter will be invoked if a pointer is already owned.
       */
      void
      reset(pointer __p = pointer()) noexcept
      {
	using std::swap;
	swap(_M_t._M_ptr(), __p);
	if (__p != pointer())
	  get_deleter()(__p);
      }

      /// Exchange the pointer and deleter with another object.
      void
      swap(unique_ptr& __u) noexcept
      {
	using std::swap;
	swap(_M_t, __u._M_t);
      }

      // Disable copy from lvalue.
      unique_ptr(const unique_ptr&) = delete;
      unique_ptr& operator=(const unique_ptr&) = delete;
  };

```

