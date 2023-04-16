#ifndef __THREAD_POOL__
#define __THREAD_POOL__

#include <boost/asio.hpp>
#include <boost/asio/io_context.hpp>
#include <boost/asio/strand.hpp>
#include <boost/thread/thread.hpp>

int test(int a , int b) {
	return a + b;
}

class Worker
{

private:
    template<typename F, typename ... ARGS>
    auto RvalueBind(F && func, ARGS && ... args)
    {
        return [func = std::forward<F>(func),
                args = std::make_tuple( std::forward<ARGS>(args)... )]() 
                mutable 
                {
                    return std::apply( func, std::move(args) );
                };
    }

	constexpr static size_t notification_thread_num = 2;
	std::unique_ptr<boost::asio::thread_pool> m_thread_pool;

public:
	void start() {
		m_thread_pool = std::make_unique<boost::asio::thread_pool>(notification_thread_num);
		

	}
	
	void stop() {
	    m_thread_pool->join();
	}
	
	void run () {
		int a , int b; 
		auto callback = []() { return true; }
		boost::asio::post(
			*m_thread_pool, RvalueBind(std::move(test), std::move(a), std::move(b), std::move(callback))
		);
	}
};



#endif